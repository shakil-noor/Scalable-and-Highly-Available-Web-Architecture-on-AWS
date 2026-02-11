# Scalable Web Application Deployment on AWS

## 📌 Project Overview

This project demonstrates the design and deployment of a highly available and scalable web application architecture on AWS. The infrastructure uses an Application Load Balancer (ALB) to distribute traffic across EC2 instances managed by an Auto Scaling Group. The system dynamically scales based on CloudWatch metrics to maintain performance and optimize cost. The architecture is deployed across multiple Availability Zones to ensure fault tolerance and high availability. Monitoring and alerting are configured using Amazon CloudWatch.

---

## 🏗 Architecture Summary

The solution includes:

* IAM user configuration for administrative access
* Security group isolation between layers
* Launch Template–based EC2 provisioning
* Application Load Balancer (ALB)
* Target Group health monitoring
* Auto Scaling Group across multiple Availability Zones
* CloudWatch alarms for dynamic scaling
* SNS notifications for operational alerts

Traffic flow:

```
User → Internet Gateway → ALB → Target Group → Auto Scaling EC2 Instances
```

Monitoring flow:

```
CloudWatch Metrics → Alarm Trigger → SNS Notification
```

---

## 🧭 Architecture Diagram

![AWS Architecture](Scalable Web Application Architecture on AWS.png)

---

## 🧰 AWS Services Used

* IAM
* VPC
* EC2
* Launch Templates
* Application Load Balancer
* Target Groups
* Auto Scaling Groups
* CloudWatch
* SNS

---

## 🚀 Deployment Steps (High Level)

---

### 🔐 Step 1: C**reate an AWS IAM user with `AdministratorAccess` policy:**

### 1. **Sign in to AWS Console**

- Go to: https://console.aws.amazon.com/
- Log in as a user with permissions to manage IAM users (e.g., root or admin user).

### 2. **Open IAM Service**

- In the AWS Console, search for and click on **IAM (Identity and Access Management)**.

### 3. **Create a New User**

- In the left sidebar, click **Users**.
- Click the **"Add users"** button.

### 4. **Set User Details**

- **User name**: Enter a username (e.g., `admin-user`).
- **Access type**:
    - ✅ Check **"Access key - Programmatic access"** (for CLI/API).
    - ✅ Check **"Password - AWS Management Console access"** (for web console login).
- Optionally set a custom password or let AWS generate one.

### 5. **Set Permissions**

- Choose **"Attach policies directly"**.
- In the search bar, type: `AdministratorAccess`.
- ✅ Check the box next to **AdministratorAccess** (Full access to AWS resources).

### 6. **Add Tags** (optional)

- Add key-value tags if needed (e.g., `Key: Environment`, `Value: Dev`), then click **Next**.

### 7. **Review and Create**

- Review the settings and click **Create user**.

---

### ✅ **After Creation**

- You will see a success message.
- Download or copy the:
    - **Console login URL**
    - **Username**
    - **Password**
    - **Access key ID and secret** (if programmatic access was enabled)

---

## 🛡️ Step 2: Create Security Groups

### 1. **Security Group for ALB**

1. Go to **VPC** > **Security Groups** > **Create security group**.
2. Name: `ALB-SG`
3. Description: Security group for ALB
4. VPC: Select your desired VPC
5. Inbound rules:
    - **HTTP (80)** – Source: `0.0.0.0/0` (for public access)
    - (Optional) **HTTPS (443)** – Source: `0.0.0.0/0`
6. Outbound rules:
    - Leave default (All traffic allowed)

> 🔐 This security group allows users to access the ALB publicly over HTTP/HTTPS.
> 

### 2. **Security Group for EC2 (Launch Template)**

1. Create another security group.
2. Name: `EC2-SG`
3. Description: Security group for instances in launch template
4. VPC: Same as above
5. Inbound rules:
    - **HTTP (80)** – Source: `ALB-SG` (this allows only the ALB to talk to instances)
    - (Optional) **SSH (22)** – Source: your IP (for management/debugging)
6. Outbound rules:
    - Leave default

> 🔒 This ensures only the ALB (not public traffic) can reach the instances.
> 

---

## 🧰 Step 3: Create Launch Template

1. Go to **EC2** > **Launch Templates** > **Create launch template**
2. Name: `MyLaunchTemplate`
3. Template version description: `v1`
4. AMI: Choose a base Amazon Machine Image (e.g., Amazon Linux)
5. Instance Type: Choose desired type (e.g., `t2.micro`)
6. Key Pair: Select an existing key pair or create a new one
7. Network Settings:
    - Security group: Select `EC2-SG` created above
8. Storage: Leave default or modify as needed
9. Advanced settings (optional): Add user data to install web server or app
    
    ```bash
    sudo yum update -y
    sudo yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Hello from Launch Template</h1>" > /var/www/html/index.html
    ```
    
10. Click **Create launch template**

---

## 🎯 Step 4: Create Target Group

1. Go to **EC2 > Target Groups > Create target group**
2. Settings:
    - Type: `Instances`
    - Name: `MyTargetGroup`
    - Protocol: HTTP, Port: 80
    - Health Check Path: `/`
3. Click **Next**, and **Skip** registering targets (it’ll auto-register via Auto Scaling group later)

---

## 🌐 Step 5: Create Application Load Balancer (ALB)

1. Go to **EC2 > Load Balancers > Create Load Balancer**
2. Select **Application Load Balancer**
3. Fill:
    - Name: `MyALB`
    - Scheme: `Internet-facing`
    - IP address type: `IPv4`
    - Listeners: HTTP (80)
    - Availability Zones: Select 2+ subnets in your VPC
    - Security Group: `ALB-SG`
4. Routing:
    - Target group: `MyTargetGroup`
5. Click **Create Load Balancer**

---

## 📈 Step 6: Create Auto Scaling Group (ASG)

1. Go to **EC2 > Auto Scaling Groups > Create Auto Scaling Group**
2. Settings:
    - Name: `MyASG`
    - Launch Template: `MyLaunchTemplate`
    - Version: `Default`
    - Subnets: 2+ in same VPC
3. Attach to Load Balancer:
    - Choose ALB
    - Target Group: `MyTargetGroup`
4. Group Size:
    - Desired capacity: `1` (or more)
    - Min: `1`, Max: `3`
5. Scaling Policies: Leave default (or add later)
6. Finish setup and create ASG

---

## 🖥️ Step 7:  Launch EC2 Instance from a Launch Template (Without Auto Scaling Group)

1. Go to the **EC2 Dashboard** in the AWS Console.
2. On the left sidebar, click **Instances**.
3. Click the **"Launch instance"** button.
4. Now click the tab **"Launch instance from template"**.
5. Choose your template:
    - **Launch template**: Select the one you created (e.g., `MyLaunchTemplate`)
    - **Version**: Choose "Default" or a specific version (e.g., `v1`)
6. Review the pre-filled settings from the template (AMI, instance type, key pair, security group, etc.)
7. Make sure the **network/subnet** is selected properly under "Network settings".
8. Click **Launch instance**.

## After Launching

- Go to **EC2 > Instances** to see the instance being initialized.
- If your User Data installed a web server, you can check it:
    - Copy the **Public IPv4** of the instance.
    - Open a browser and go to `http://<public-ip>` to see your web page.

## Access the instance from local machine and test by install web app

- Access the EC2 instance form the terminal
- Clone a simple html website form the git: git clone https://github.com/bhavukm/webapp-asg-alb.git
- copy the folder  “webapp-asg-alb” at this folder /var/www/html
- Start and enable the apache
    - systemctl start httpd
    - systemctl enable httpd
    - systemet status httpd
- Open a browser and go to `http://<public-ip>` to see your web page.

---

### 🔄 Step 8: Create an **Amazon Machine Image (AMI)** from an **EC2 instance**

1. Go to **EC2 > Instances**
2. **Select Your Instance**:
    - In the left menu, click **Instances**.
    - Select the EC2 instance you want to create the image from.
3. **Create Image**:
    - Click the **Actions** dropdown.
    - Go to **Image and templates** → **Create image**.
4. **Configure Image Settings**:
    - **Image name**: Enter a name for the image.
    - **Image description** (optional): Describe what the image is for.
    - Choose whether to **reboot the instance** during the image creation (reboot ensures a clean state).
    - Optionally include/exclude additional volumes.
5. **Create Image**:
    - Click **Create image**.
    - Go to **AMIs** (under Images in the left panel) to see the status of your new image.

---

### 🧬 Step 9: Create New Launch Template Version

1. **Go to EC2 → Launch Templates**:
    - Find your existing launch template.
2. **Actions → Create new version**:
    - Click on your launch template name.
    - Click the **“Actions”** dropdown.
    - Select **“Create new version”**.
3. **Update the AMI**:
    - Under **Amazon Machine Image (AMI)**, click **Browse AMIs**.
    - Select the new AMI you just created.
    - You can also update other settings (like instance type, key pair, etc.) if needed.
4. **Create Template Version**:
    - Scroll down and click **Create launch template version**.
5. **(Optional) Set as Default Version**:
    - Back in the launch template view, click **Actions** → **Set default version**.
    - Choose your newly created version.

Now this launch template has new AMI version

---

### 🔁 Step 10: E**nsure the new launch template version is attached to Auto Scaling Group (ASG)**

1. **Go to EC2 → Auto Scaling Groups**.
2. Select your **Auto Scaling Group**.
3. Click **Edit** (top-right corner) on **details tab**.
4. Under **Launch Template**, make sure:
    - The correct **launch template name** is selected.
    - Under **Version**, select:
        - **Latest (default)** – if you've set the new version as default.
        - Or **Specific version** – and select the new version manually.
5. Click **Update** at the bottom.

---

## 📊 Step 11: Configure Dynamic Scaling with CloudWatch

### **1. Go to the EC2 Dashboard**

- Open the AWS Console: https://console.aws.amazon.com/ec2
- In the left menu, scroll down to **Auto Scaling Groups**.
- Click your **Auto Scaling Group (ASG)** name.

### **2. Create a Scaling Policy**

- Inside your ASG view, go to the **"Automatic scaling"** tab.
- Under **"Dynamic scaling policies"**, click **“Add policy”**.

### Fill out:

- **Policy type**: `Step scaling`
- **Policy name**: e.g., `scale-out-policy`
- **CloudWatch alarm**: Click **“Create an alarm”**

### **3. Create CloudWatch Alarm (inside scaling policy form)**

In the popup:

- **Metric type**: `Average CPU utilization`
- **Threshold type**: `Static`
- **Whenever CPU is...** `>= 70`
- **For...** `2 consecutive periods`
- **Period**: `1 minute` or `5 minutes` (based on your preference)

Click **“Create alarm”** – it will attach to the scaling policy automatically.

---

### 📬 Step 12: Create SNS Topic for Alerts

If you didn't create one in the previous step:

1. Open the **SNS Console**.
2. Click **Topics > Create topic**.
3. Choose **Standard**.
4. Add **subscriptions** to receive alerts (e.g., your email).
5. Set a name (e.g., `ScaleUpAlerts`) and click **Create topic**.

### **Define Scaling Action**

Back in the scaling policy form:

- **Take the action**: `Add 1 instance`
- Optionally, configure cooldown settings.

Click **"Create"** to finalize the policy.

### ✅ (Optional) Create a Scale-In Policy

Repeat the above steps but:

- Set **CPU threshold ≤ 20**
- Use **"Remove 1 instance"** as the scaling action.

---

## 🧪 Step 13: Simulate CPU Load (Install `stress` Tool)

Ensure your Launch Template or Launch Configuration includes **user data** to install `stress` automatically.

Edit the **Launch Template** or create a new one with this user data:

### For Amazon Linux 2:

```bash
yum update -y
yum install -y epel-release
yum install -y stress
stress --cpu 2 --timeout 600
```

### For Ubuntu:

```bash
apt-get update
apt-get install -y stress
stress --cpu 2 --timeout 600
```

> This stresses the CPU for 10 minutes. You can adjust the --timeout or --cpu value.
> 

## Update ASG to Use the New Launch Template ( Follow step 10)

1. Go to **EC2 > Auto Scaling Groups**.
2. Select your ASG.
3. Click **Edit** and change the **Launch Template version** to the latest one with the `stress` setup.
4. **Save changes**.
5. Manually **terminate an instance** in the group to force it to launch a new one using the updated template.

## Monitor CPU Usage in CloudWatch

1. Go to **CloudWatch Console > Metrics**.
2. Browse to:
    
    ```
    Auto Scaling > Per Auto Scaling Group Metrics > CPUUtilization
    ```
    
3. Watch for spikes in CPU (should rise to 80–100% depending on how many `-cpu` threads you run).

## Check for Scaling Activity

1. Go to **EC2 > Auto Scaling Groups**.
2. Select your group, then click the **Activity** tab.
3. Look for entries like:
    - "Launching a new EC2 instance"
    - "Terminating EC2 instance"
    - Reason: e.g., "At 2025-04-11T13:00Z an instance was launched due to CPUUtilization..."

## 🔔 Optional: Check SNS Notification

If you connected your alarm to an **SNS topic**, check your email (or other endpoint) to confirm the notification was sent when the alarm triggered.

## 🧪 Tip: Trigger Scaling Quickly

Use a more intense stress command to spike CPU faster:

```bash

stress --cpu 4 --timeout 900
```

Or run multiple in parallel for multi-core instances.

---

## ✅ Key Learning Outcomes

* Understanding multi-AZ high availability design
* Practical Auto Scaling configuration
* Load balancing traffic distribution
* Monitoring system metrics
* Infrastructure version management via AMIs
* Security group layering strategy
* Alert-driven operations
