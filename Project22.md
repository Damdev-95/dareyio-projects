
## DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER

* I used terraform to deploy the EKS cluster ,kindly find my github repo 
* Installing  kubectl CLI 

```
$ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
kubectl version --short --client
  Client Version: v1.21.2-13+d2965f0db10712
```
![image](https://user-images.githubusercontent.com/71001536/176284273-698bfa24-9d8c-4c06-be01-0b0ad1c95edb.png)


# Deploying a random Pod

* Create a Pod `nginx-pod.yaml` manifest on your master node

```
 apiVersion: v1
 kind: Pod
 metadata:
   name: nginx-pod
   labels:
     app: nginx-pod
 spec:
   containers:
   - name: nginx
     image: nginx:latest
     ports:
     - containerPort: 80
       protocol: TCP
```

*  Apply the manifest with the help of kubectl

```
kubectl apply -f nginx-pod.yaml
```

*  Get an output of the pods running in the cluster
```
kubectl get pods
```
![image](https://user-images.githubusercontent.com/71001536/176285203-e7034e8d-c4c2-4483-8987-4d822ecad596.png)

* To check optional  fields given by kubernetes after deployed the resource, run below command
```
kubectl get pod nginx-pod -o yaml 
or
kubectl describe pod nginx-pod
```
![image](https://user-images.githubusercontent.com/71001536/176285770-1fe56ecb-cbe1-48c9-b411-7c3e4768f833.png)

## REACHING THE WEB SERVER(NGINX) FROM THE WEB BROWSER

Now you have a running Pod. What’s next?
*  We need another Kubernetes object called Service to accept our request and pass it on to the Pod so we can access it through the browser

```
kubectl get pod nginx-pod  -o wide 
```
![image](https://user-images.githubusercontent.com/71001536/176286169-b6239166-3eb2-4985-b3d2-eb9b7ad2b2ae.png)

Let us try to access the Pod through its IP address from within the K8s cluster. To do this,

1. We need an image that already has curl software installed. You can check it out [here](https://hub.docker.com/r/dareyregistry/curl)
`dareyregistry/curl`
2. Run kubectl to connect inside the container
```
kubectl run curl --image=dareyregistry/curl -i --tty
```
3. Run curl and point to the IP address of the Nginx Pod (Use the IP address of your own Pod) 
>  `curl -v 10.0.0.163:80`


### Let us create a service to access the Nginx Pod

1. Create a Service yaml manifest file:
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-pod 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

2. Create a nginx-service resource by applying your manifest
```
kubectl apply -f nginx-service.yaml
```

3. Check the created service `kubectl get service`

4. To access the app since there is no public IP address, we can leverage kubectl's port-forward functionality.
```
kubectl  port-forward svc/nginx-service 8089:80
```
!![image](https://user-images.githubusercontent.com/71001536/176304382-c443e57b-bb34-4019-bd70-66f09cd489a5.png)

Unfortunately, this will not work quite yet. Because there is no way the service will be able to select the actual Pod it is meant to route traffic to. If there are hundreds of Pods running, there must be a way to ensure that the service only forwards requests to the specific Pod it is intended for.

- To make this work, you must reconfigure the Pod manifest and introduce labels to match the selectors key in the field section of the service manifest.

1. Update the Pod manifest with the below and apply the manifest:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod  
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
```
Apply the manifest with `kubectl apply -f nginx-pod.yaml`

2. Run kubectl port-forward command again
```
kubectl  port-forward svc/nginx-service 8089:80
```
* Then go to your web browser and enter localhost:8089 – You should now be able to see the nginx page in the browser.

## UNDERSTANDING THE CONCEPT

## Expose a Service on a server’s public IP address & static port

A Node port service type exposes the service on a static port on the node’s IP address. NodePorts are in the 30000-32767 range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).

Update the nginx-service yaml to use a NodePort Service.
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30080
```

* To access the service, you must:

* Allow the inbound traffic in your EC2’s Security Group to the NodePort range 30000-32767
* Get the public IP address of the node the Pod is running on, append the nodeport and access the app through the browser.

![image](https://user-images.githubusercontent.com/71001536/176379494-33f6cae5-204e-41f7-af0f-89356a91fe0a.png)

## USING AWS LOAD BALANCER TO ACCESS YOUR SERVICE IN KUBERNETES.

* To get the experience of this service type, update your service manifest and use the LoadBalancer type. Also, ensure that the selector references the Pods in the replica set.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
```

* Run the configuration 

`kubectl apply -f nginx-service.yaml`

* Get the newly created service 

`kubectl get service nginx-service`

![image](https://user-images.githubusercontent.com/71001536/176380097-f4478fb0-0117-4a81-9d3c-34ce90f2988e.png)

* An ELB resource will be created in your AWS console.
* Get the output of the entire yaml for the service. You will some additional information about this service in which you did not define them in the yaml manifest. Kubernetes did this for you.
```
kubectl get service nginx-service -o yaml
```

![image](https://user-images.githubusercontent.com/71001536/176380308-5a637449-85db-43dc-b0af-9125155215b0.png)

* Copy and paste the load balancer’s address to the browser, and you will access the Nginx service

![image](https://user-images.githubusercontent.com/71001536/176399133-85519648-d429-479c-bb3d-f2013d083ada.png)


## CREATE A REPLICA SET
- Let us create a rs.yaml manifest for a ReplicaSet object
```
#Part 1
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
    
#Part 2
  template:
    metadata:
      name: nginx-pod
      labels:
         app: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
          protocol: TCP
```
```
kubectl apply -f rs.yaml

kubectl get pods
```

## I had issue at using replica set 

![image](https://user-images.githubusercontent.com/71001536/176391818-575ca0d6-7726-4ac7-9f13-2c9beef2e38d.png)

## It was resolved by specifying the match label

![image](https://user-images.githubusercontent.com/71001536/176392234-36d4ee94-55a5-4bae-bf3f-5cbe255bc97e.png)

![image](https://user-images.githubusercontent.com/71001536/176392422-5c6af060-2988-4b78-85eb-d40eb137505c.png)

Scaling Replicaset up and down:

**Imperative:**

* We can now easily scale our ReplicaSet up by specifying the desired number of replicas in an imperative command, like this:
```
kubectl scale rs nginx-rs --replicas=5
```
![image](https://user-images.githubusercontent.com/71001536/176393580-e0ed000b-042d-4b94-8609-393b9f80c2d2.png)

**Declarative:**

- Declarative way would be to open our `rs.yaml manifest`, change desired number of replicas in respective section


### USING DEPLOYMENT CONTROLLERS
**Do not Use Replication Controllers – Use Deployment Controllers Instead**

Officially, it is highly recommended to use Deplyments to manage replica sets rather than using replica sets directly.

Let us see Deployment in action.
1. Delete the ReplicaSet
```
kubectl delete rs nginx-rs
```

2. Create `deployment.yaml` manifest
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
```

`kubectl apply -f deployment.yaml`

**Run commands to get the following**

1. Get the Deployment
```
kubectl get deployment nginx-deployment
```
![image](https://user-images.githubusercontent.com/71001536/176394741-bd548f96-1f87-414a-8334-f441df227019.png)

2. Get the ReplicaSet
```
kubectl get rs <name of replicasSet>
```
![image](https://user-images.githubusercontent.com/71001536/176395126-0cf787bf-41ad-4307-b1cd-675e851fa7c9.png)

3. Get the Pods
```
kubectl get pod nginx-pod
```
4. Scale the replicas in the Deployment to 15 Pods
```
kubectl scale rs <name of replica> --replicas=<number of replica>
```
![image](https://user-images.githubusercontent.com/71001536/176396381-83b6d06b-05f5-4e1d-9ab1-caae08872863.png)


5. Exec into one of the Pod’s container to run Linux commands
```
kubectl exec <name of pod> -it bash
```
![image](https://user-images.githubusercontent.com/71001536/176396775-aee31dd0-aac8-4c33-974a-7fdfccca77a5.png)

*  List the files and folders in the Nginx directory `ls -ltr /etc/nginx/`

![image](https://user-images.githubusercontent.com/71001536/176396894-a9271664-bc17-49db-a525-7d957bf95263.png)

* Check the content of the default Nginx configuration file `cat  /etc/nginx/conf.d/default.conf `

![image](https://user-images.githubusercontent.com/71001536/176397121-8895e6e0-328f-4f81-995c-b94f16a695a6.png)

![image](https://user-images.githubusercontent.com/71001536/176397475-708a7fa4-1d71-4006-ab17-5d291d7fe59e.png)

## PERSISTING DATA FOR PODS

Required to update the content of the index.html file inside the container, and the Pod dies, that content will not be lost since a new Pod will replace the dead one.

Let us try that:

1. Scale the Pods down to 1 replica.
2. Exec into the running container 
3. Install vim so that you can edit the file
4. Update the content of the file and add the code below `/usr/share/nginx/html/index.html`
5. Check the browser
6. Now, delete the only running Pod
7. Refresh the web page – You will see that the content you saved in the container is no longer there. That is because Pods do not store data when they are being recreated – that is why they are called ephemeral or stateless
