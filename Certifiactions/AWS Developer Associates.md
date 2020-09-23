#### Regions

IAM = Identity & Access Management

IAM Federation= Big Companies connect their user base to AWS and user then can login using their company credentials. Microsoft Active Directory is an example of this and. Identity federation uses SAML standard.

#### Security Groups

- Security Group can be attached to multiple instance. Better to use separate security group for in-bound and out-bound rules.
- By default all inbound traffic is blocked and all out-bound traffic is authorized. If application is not accessible/timed out, then probably security group issue. But if "connection refused" then application error or app launch issue. 
- It's better to maintain separate security group for SSH access.

#### Elastic IP

- One account can have 5 Elastic IP. We can ask AWS to increase that.
- Overall, try to avoid Elastic IP. Using this reflects poor architectural design. Instead use random IP and DNS or LB.

EC2 User Data- AMI

- Run once when instance boots up. Used for installing update, installing software, download, anything we can think of!! 
- EC2 User Data script runs with root user.
- Custom AMI is build for specific AWS region. 

ENI- Elastic Network Interface

- Logical component. Represents virtual network card. It will have one primary private IPV4, and can have more secondary private IPV4. 
- Can attach to instances, and can move to other instances. Bound to AZ.

Load Balancers

- 3 Types. Classic LB, Application LB (*Layer 7)*, Network LB(*Layer 4)*. Managed. Checks if "/health" is returning 200 or not!
- ELB will connect with multiple Target Group. Stickiness can enabled on target group level. Same instance will have same request.
- ALB supports HTTP, HTTPS, WebSocket.  Application server does not see client IP directly since LB forwarded the request to application server. True IP of client is inserted in the **header X-Forwarded-For** and the protocol is **X-Forwarded-Proto**
- ALB: Load balance to multiple http application **across machine** (target groups), multiple application on **same machine**(containers). **Supports redirect** (from *http to https* or vice versa for example). Great fit for *Microservices/containers*.
- ALB **redirect** to different *target groups*: can be done based on **hostname**(abc.com|def.com), based on **path in url** (abc.com/user| abc.com/post), based on **query string** (/user?id=123&order=false). ALB can return **response** based on **rules** if configured.
- NLB: Handles Millions of traffic per seconds. Support for Elastic and Static IP. No HTTP, only TCP, TLS, UDP etc. NLB is used for **extreme** performance. Not **free**. Less Latency. **100 ms (NLB), 400 ms(ALB)**
- NLB: is not attached with any Security Group. So, instances see traffic comes from the outside. So, need to allow instance security group TCP, UDP and open to all (0.0.0.0/0)
- LB Stickiness: Stickiness works for classic LB & ALB. Stickiness uses "cookie" with an expiration date user can control. For this, make sure user doesn't lose session data. Enabling stickiness can cause imbalance to load of instances.
- LB can scale but not instantaneously! Need to contact AWS for warmup. **Multi AZ**.
- 4XX = client errors, 5XX= application errors, LB error 503= at capacity or no registered target. If LB cannot connect to application, security group issue probably.
- Cross AZ balancing. Classic is not enabled by default, but can be enabled with no charge. ALB is enabled by default with no charge. But NLB is disabled by default, and can be enabled with charge.

ASG - Auto Scaling Group

- ASG scales based on CloudWatch Alarms. ASG is free, u need to pay for underlying resources only. **Multi AZ**.
- ASG use launch configuration and we can update ASG by providing new Launch configuration. IAM role attached to an ASG is assigned to EC2 instances.
- ASG will have min & max instance policy. It can restart an instance if it terminates for any reason or unhealthy.

EBS Volume

- Its a network drive. *1 EBS* can be attached to *1 instanc*e at a time. *Locked* to an (AZ) Avaibility Zone. EBS volume in us-east-1a	 cannot be attached to us-east-1b. So, to migrate EBS volume we need to *backup snapshot and then re-create* in other AZ.
- We can *resize* EBS volume. Can take snapshot. For 100GB EBS volume with 5GB data, snapshot will contain only data part, 5GB.
- **EBS encryption**. **1**- Data inside volume is encrypted. **2**-All data flight moving between instance & volume is encrypted. **3**- Snapshots are encrypted. **4**- Volumes created from snapshot is encrypted. 
- Instance Store. Some instance does not have EBS volume rather instance store which is *physically attached*. Pros is better performance. However, cons is store will be **lost a**t termination, *can't* be **resized**, user needs to manage.

Multi AZ

- Auto Scaling Group
- Load Balancers