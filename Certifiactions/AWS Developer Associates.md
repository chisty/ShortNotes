#### Regions

IAM = Identity & Access Management

IAM Federation= Big Companies connect their user base to AWS and user then can login using their company credentials. Microsoft Active Directory is an example of this and. Identity federation uses SAML standard.

#### Security Groups

- Security Group can be attached to multiple instance
- Better to use separate security group for in-bound and out-bound rules.
- By default all inbound traffic is blocked and all out-bound traffic is authorised. If application is not accessible/timed out, then probably security group issue. But if "connection refused" then application error or app launch issue. 

#### Elastic IP

- Max 5 items. 

EC2 User Data

- Run once when instance boots up. Used for installing update, installing software, download, anything we can think of!! 
- EC2 User Data script runs with root user.
- Custom AMI is build for specific AWS region. 

Load Balancers

- 3 Types. Classic LB, Application LB (*Layer 7)*, Network LB(*Layer 4)*. Managed. Checks if "/health" is returning 200 or not!
- ELB will connect with multiple Target Group. Stickiness can enabled on target group level. Same instance will have same request.
- ALB supports HTTP, HTTPS, WebSocket.  Application server does not see client IP directly since LB forwarded the request to application server. True IP of client is inserted in the **header X-Forwarded-For.** 
- NLB: Handles Millions of traffic per seconds. Support for Elastic and Static IP.
- Less Latency. **100 ms (NLB), 400 ms(ALB)**
- LB can scale but not instantaneously! Need to contact AWS for warmup
- 4XX = client errors, 5XX= application errors, LB error 503= at capacity or no registered target. If LB cannot connect to application, security group issue probably.

ASG - Auto Scaling Group

- ASG scales based on CloudWatch Alarms. ASG is free, u need to pay for underlying resources only.
- ASG use launch configuration and we can update ASG by providing new Launch configuration. IAM role attached to an ASG is assigned to EC2 instances.
- ASG will have min & max instance policy. It can restart an instance if it terminates for any reason or unhealthy.

EBS Volume

- Its a network drive. *1 EBS* can be attached to *1 instanc*e at a time. *Locked* to an (AZ) Avaibility Zone. EBS volume in us-east-1a	 cannot be attached to us-east-1b. So, to migrate EBS volume we need to *backup snapshot and then re-create* in other AZ.
- We can *resize* EBS volume. Can take snapshot. For 100GB EBS volume with 5GB data, snapshot will contain only data part, 5GB.
- **EBS encryption**. **1**- Data inside volume is encrypted. **2**-All data flight moving between instance & volume is encrypted. **3**- Snapshots are encrypted. **4**- Volumes created from snapshot is encrypted. 
- Instance Store. Some instance does not have EBS volume rather instance store which is *physically attached*. Pros is better performance. However, cons is store will be **lost a**t termination, *can't* be **resized**, user needs to manage.