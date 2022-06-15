## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

Introducing Backend on S3
Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc.
Take a peek into what the states file looks like. It is basically where terraform stores all the state of the infrastructure in json format.

So far, we have been using the default backend, which is the local backend – it requires no configuration, and the states file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.

The second problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on your computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely other DevOps team members. There are plenty of different standard backends supported by Terraform that you can choose from. Since we are already using AWS – we can choose an S3 bucket as a backend.

Another useful option that is supported by S3 backend is State Locking – it is used to lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service – DynamoDB.

![image](https://user-images.githubusercontent.com/71001536/173785054-0a309422-27c9-4158-9d99-071f6c59a201.png)

![image](https://user-images.githubusercontent.com/71001536/173804586-1dde4808-5b9f-4b59-983a-8e1cac45f4da.png)

![image](https://user-images.githubusercontent.com/71001536/173806281-98ac4b43-2eac-409c-8f39-134421b9a735.png)

![image](https://user-images.githubusercontent.com/71001536/173808060-4bbf1634-125f-4178-aa2f-c38c692ef315.png)

# Final Terraform plan 
![image](https://user-images.githubusercontent.com/71001536/173808389-e11253f6-06d2-468f-8c45-0a86ea9743cd.png)

