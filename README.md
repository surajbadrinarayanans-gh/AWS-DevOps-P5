# AWS DevOps Project:

---

#### Project Introduction:

I built a secure, private-first AWS infrastructure with controlled public access using a reverse proxy.

Implemented persistent storage with EBS and enabled live scaling without downtime.

Designed a serverless monitoring pipeline using S3, Lambda, EventBridge, and CloudWatch for automated observability.

#### Project Architecture:

![1773826294364](image/README/1773826294364.png)

---

#### Phase 1 - VPC, Subnets Setup:

![1773803887094](image/README/1773803887094.png)

Creating the VPC

    ![1773803994106](image/README/1773803994106.png)

Creating public and private subnets.

Public subnet for:

- Jump Host to access private server.
- NAT Gateway for external interent access to private instance.

Private subnet for internal application logic, disk usage monitoring and logs storage.

![1773804041085](image/README/1773804041085.png)

Creating the Internet Gateway for the VPC.

![1773804436236](image/README/1773804436236.png)

Creating Elastic IP for NAT gateway.

![1773804502082](image/README/1773804502082.png)

Creating the NAT gateway and deploying it in the public subnet.

![1773804640149](image/README/1773804640149.png)

Attaching the public subnet to the internet gateway through public subnet route table.

![1773804725797](image/README/1773804725797.png)

Attaching the private subnet to NAT Gateway through private subnet route table.

---

#### Phase 2 - Server deployment:

![1773805937070](image/README/1773805937070.png)

Creating public and private instance for jump host and internal application logic respectively.

![1773807952980](image/README/1773807952980.png)

Securely connecting to the public instance using keypair and ssh method.

![1773807714384](image/README/1773807714384.png)

Allowing the IP of public instance (jump host) in the private instance security group so that only the bastion host can access the private instance.

![1773807749070](image/README/1773807749070.png)

Verifying the internet outbound connectivity on the private instance.

---

#### Phase 3 - Persistent Storage for the Application Logic:

![1773808027885](image/README/1773808027885.png)

Inspecting the storage blocks initially right after creating and logging into the instances.

![1773808415596](image/README/1773808415596.png)

Creating a new EBS volume for persistent storage.

![1773808485541](image/README/1773808485541.png)

Attaching the volume to the private instance.

![1773808514339](image/README/1773808514339.png)

Inspecting the storage bloacks after the attachment of EBS volume. (nvme1n1 is attached successfully)

![1773808599910](image/README/1773808599910.png)

Creating the dick partition to actually use the storage for the application.

![1773808695783](image/README/1773808695783.png)

Attaching a file system for the partition and mounting it to a directory.

![1773808712047](image/README/1773808712047.png)

Verifying the mount point for the newly created partition.

![1773808832327](image/README/1773808832327.png)

Inspecting the UUID for persistent storage of the data directory because when the instance stops or reboots, the data will be lost.

![1773808814564](image/README/1773808814564.png)

Adding the mount point permanently in /etc/fstab.

![1773808998923](image/README/1773808998923.png)

Unmounting the data directory and remounting it to verify persistance.

---

#### Phase 4 - Resizing Without Downtime:

![1773809550771](image/README/1773809550771.png)

Modifying the size in EBS console.

![1773809791542](image/README/1773809791542.png)

Verifying if the storage is increased.

![1773809964969](image/README/1773809964969.png)

But the mount point store is not increased because it must be resized explicitely.

![1773809951353](image/README/1773809951353.png)

Verifying the increase in storage of the mount point without downtime.

---

#### Phase 5 - Application:

![1773813044401](image/README/1773813044401.png)

Installing and verifying the status of nginx service.

![1773813743663](image/README/1773813743663.png)

- Creating the application home page inside the /data directory.
- Editing the default nginx startup path to /data.
- Restarting the service to see the appropriate changes.

![1773813713668](image/README/1773813713668.png)

The Service can be accessed successfully from the bastion host.

![1773814010124](image/README/1773814010124.png)

Install nginx in bastion host and modifying the config file so that it acts as a reverse proxy for the private instance application.

![1773814332867](image/README/1773814332867.png)

![1773814418128](image/README/1773814418128.png)

Accessing the application via a brower in my personal laptop.

---

#### Phase 6 - Severless Monitoring:

![1773817016739](image/README/1773817016739.png)

Creating a S3 bucket that stores the disk usage of the data directory.

![1773817104588](image/README/1773817104588.png)

Creating role for lambda function and attaching necessary policies.

![1773817214371](image/README/1773817214371.png)

Creating the lambda function that processes content from S3 and creates CloudWatch Logs.

![1773817804967](image/README/1773817804967.png)

Creating an Event Bridge that triggers lambda function every 5 minutes.

![1773818240659](image/README/1773818240659.png)

![1773818202413](image/README/1773818202413.png)

Creating EC2 role and attaching necessary policies so that it can push the disk metrics to S3 bucket.

![1773818296742](image/README/1773818296742.png)

Attaching the role to private EC2 instance.

![1773818477276](image/README/1773818477276.png)

Inspecting the disk usage under /data directory.

![1773818450250](image/README/1773818450250.png)

Creating a script file that writes the disk usage to S3 bucket.

![1773818678157](image/README/1773818678157.png)

Installing the AWS CLI and running the script.

![1773818705129](image/README/1773818705129.png)

The file is successfully written into the S3 bucket.

![1773819151323](image/README/1773819151323.png)

1% disk usage is stored in the file. This is taken by the lambda function to generate cloudwatch logs that can be used later to create notifications or alerts.

---

#### Phase 7 - Deletion of Resouces:

![1773819447510](image/README/1773819447510.png)

![1773819481995](image/README/1773819481995.png)

![1773819601368](image/README/1773819601368.png)

![1773819683383](image/README/1773819683383.png)

Delete Lambda Function

![1773819765515](image/README/1773819765515.png)

![1773819806832](image/README/1773819806832.png)

![1773819858159](image/README/1773819858159.png)

![1773819899307](image/README/1773819899307.png)

---
