# PERSISTING DATA IN KUBERNETES

Now we know that containers are stateless by design, which means that data does not persist in the containers. Even when you run the containers in kubernetes pods, they still remain stateless unless you ensure that your configuration supports statefulness.

To achieve statefuleness in kubernetes, you must understand how volumes, persistent volumes, and persistent volume claims work.

## sTEPS FOR CREATING VOLUME ON AWS 

* In your AWS console, head over to the EC2 section and scroll down to the Elastic Block Storage menu.

![image](https://user-images.githubusercontent.com/71001536/176399991-835125ae-df8b-4a55-ab66-9f65bad6f968.png)

* Click on Volumes
* At the top right, click on Create Volume

![image](https://user-images.githubusercontent.com/71001536/176400895-a8ba865e-335e-41a8-9b24-4c8ad673ef6d.png)

* Copy the VolumeID
![image](https://user-images.githubusercontent.com/71001536/176401540-087831d1-b8ae-4a97-864b-36d1954585a5.png)

* Update the deployment configuration with the volume spec.

```
sudo cat <<EOF | sudo tee ./nginx-pod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      volumes:
      - name: nginx-volume
        # This AWS EBS volume must already exist.
        awsElasticBlockStore:
          volumeID: "vol-0b6f7dba8efac12a6"
          fsType: ext4
EOF
```
![image](https://user-images.githubusercontent.com/71001536/176402625-41360b15-1675-4431-896e-4ee9e1394d9b.png)

* Create and edit the deployment configuration with the volume spec.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-volume
          mountPath: /usr/share/nginx/
      volumes:
      - name: nginx-volume
        # This AWS EBS volume must already exist.
        awsElasticBlockStore:
          volumeID: "vol-0b6f7dba8efac12a6"
          fsType: ext4
```

## In as much as we now have a way to persist data, we also have new problems.

1. If you port forward the service and try to reach the endpoint, you will get a 403 error. This is because mounting a volume on a filesystem that already contains data will automatically erase all the existing data. This strategy for statefulness is preferred if the mounted volume already contains the data which you want to be made available to the container


2. It is still a manual process to create a volume, manually ensure that the volume created is in the same Avaioability zone in which the pod is running, and then update the manifest file to use the volume ID. All of these is against DevOps principles because it will mean having a lot of road blocks to getting a simple thing done.

The more elegant way to achieve this is through **Persistent Volume** and **Persistent Volume claims**. In kubernetes, there are many elegant ways of persisting data. Each of which is used to satisfy different use cases. Lets take a look at the different options available.

- Persistent Volume (PV) & Persistent Volume Claim (PVC)
- configMap

## MANAGING VOLUMES DYNAMICALLY WITH PVS AND PVCS
PVs are volume plugins that have a lifecycle completely independent of any individual Pod that uses the PV. This means that even when a pod dies, the PV remains. A PV is a piece of storage in the cluster that is either provisioned by an administrator through a manifest file, or it can be dynamically created if a storage class has been pre-configured.

## A PersistentVolumeClaim (PVC) on the other hand is a request for storage. Just as Pods consume node resources, PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany or ReadWriteMany, see AccessModes).

NOTES:

1. When PVCs are created with a specific size, it cannot be expanded except the storageClass is configured to allow expansion with the allowVolumeExpansion field is set to true in the manifest YAML file. This is "unset" by default in EKS.

2. When a PV has been provisioned in a specific availability zone, only pods running in that zone can use the PV. If a pod spec containing a PVC is created in another AZ and attempts to reuse an already bound PV, then the pod will remain in pending state and report volume node affinity conflict. Anytime you see this message, this will help you to understand what the problem is.

3. PVs are not scoped to namespaces, they a clusterwide wide resource. PVCs on the other hand are namespace scoped.

* Run this command to check if you already have a storageclass in your cluster

`kubectl get storageclass`

![image](https://user-images.githubusercontent.com/71001536/176404619-ebf35ddc-4d23-42bc-87a2-07b227b7468c.png)

Approach 1

* Create a manifest file for a PVC, and based on the gp2 storageClass a PV will be dynamically created

```
apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nginx-volume-claim
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
      storageClassName: gp2
  ```
Apply the manifest file and you will get an output like below

`kubectl apply -f pv.yaml`

![image](https://user-images.githubusercontent.com/71001536/176406515-62668d46-45b6-487e-aeca-2e758dccc1fd.png)

`kubectl describe storageclass gp2`

![image](https://user-images.githubusercontent.com/71001536/176420370-9a73df36-c65a-44d1-acb9-1410d5f13ded.png)

* Then configure the Pod spec to use the PVC
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-volume-claim
          mountPath: "/tmp/damdev"
      volumes:
      - name: nginx-volume-claim
        persistentVolumeClaim:
          claimName: nginx-volume-claim
```
![image](https://user-images.githubusercontent.com/71001536/176422404-306e2dde-19b2-46e5-8ac1-d3003b012814.png)


## CONFIGMAP

Using configMaps for persistence is not something you would consider for data storage. Rather it is a way to manage configuration files and ensure they are not lost as a result of Pod replacement.

to demonstrate this, we will use the HTML file that came with Nginx. This file can be found in /usr/share/nginx/html/index.html  directory.

Lets go through the below process so that you can see an example of a configMap use case.

* Remove the volumeMounts and PVC sections of the manifest and use kubectl to apply the configuration

* port forward the service and ensure that you are able to see the "Welcome to nginx" page

* exec into the running container and keep a copy of the index.html file somewhere. For example

`kubectl exec -it nginx-deployment-98d656588-dh7qk -- bash`
  
`cat /usr/share/nginx/html/index.html`

![image](https://user-images.githubusercontent.com/71001536/176424083-0853e53a-65be-4cd1-9638-6f5e6c58ede2.png)

## Persisting configuration data with configMaps
According to the official documentation of configMaps, A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

In our own use case here, We will use configMap to create a file in a volume.

* The manifest file we look like:

```
cat <<EOF | tee ./nginx-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: website-index-file
data:
  # file to be mounted inside a volume
  index-file: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
EOF
```
* Apply the new manifest file `kubectl apply -f nginx-configmap.yaml`

![image](https://user-images.githubusercontent.com/71001536/176425023-e37c2ba5-5871-4ae7-9474-e9de544724fe.png)

![image](https://user-images.githubusercontent.com/71001536/176425746-b5a82a49-78fc-4fdd-b904-a53b088cad35.png)


* Update the deployment file to use the configmap in the volumeMounts section

```
cat <<EOF | tee ./nginx-pod-with-cm.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config
            mountPath: /usr/share/nginx/html
            readOnly: true
      volumes:
      - name: config
        configMap:
          name: website-index-file
          items:
          - key: index-file
            path: index.html
EOF
```
![image](https://user-images.githubusercontent.com/71001536/176425942-18809969-fa50-4bdd-a731-d6d2211114be.png)

![image](https://user-images.githubusercontent.com/71001536/176426284-b1cd87c1-ecee-4818-bf9d-bb7c2f777697.png)

* Update the configmap. You can either update the manifest file, or the kubernetes object directly. Lets use the latter approach this time.

* Without restarting the pod, your site should be loaded automatically.
* 
* If you wish to restart the deployment for any reason, simply use the command 
* `kubectl rollout restart deploy nginx-deployment` 
* This will terminate the running pod and spin up a new one.

![image](https://user-images.githubusercontent.com/71001536/176427611-bee8db82-986d-4efc-a9f1-ab08898118da.png)

