# AWS Flask Deployment - Auto Scaling and Load Balancer

## Project Description
 
This project aims to deploy a Flask-based Python application on AWS. The architecture will be built using Auto Scaling Groups, an Application Load Balancer, and EC2 instances. The source code will be pulled from GitHub, and the Flask application will start automatically.

## Architecture Components

* **VPC (Virtual Private Cloud)**: Creates an isolated network.
* **Public Subnet**: Network component that will host EC2 instances.
* **EC2 Instances**: Virtual machines running the Flask application.
* **Auto Scaling Group**: Scales EC2 instances based on traffic.
* **Application Load Balancer**: Distributes traffic evenly.
* **Security Groups**: Defines firewall rules.
* **GitHub**: Repository for managing the source code.

## Requirements

* AWS Account
* AWS Management Console access
* GitHub Account
* A simple Python application using Flask

## 1. Creating AWS Resources

### 1.1 Defining VPC and Subnet (AWS Management Console)

1. The default VPC and subnets will be used.

### 1.2 Configuring Security Groups

1. Security groups will be created for both EC2 instances and the ALB.

* EC2 Security Group inbound: ssh:anywhere and http:albssecuritygroup
* ALB Security Group will allow only http:anywhere.

### 1.3 Creating Load Balancer and Target Group

1. The application load balancer and target group will be created by setting the required configurations.

## 2. Creating EC2 Instances

### 2.1 Flask Setup with Launch Template

* Go to the **EC2 Dashboard** > **Launch Templates** section.
* Click the **Create Launch Template** button.
* Select **Ubuntu 24.04** as the **Amazon Machine Image (AMI)**.
* Choose **t2.micro** as the **Instance Type**.
* Add the security group you created under the **Security Group** section.
* In the **User Data** section, add the following commands:

```
#!/bin/bash
apt update -y
apt install git python3-pip python3-flask -y
cd /home/ubuntu
git clone https://github.com/your_github_repo/flask_app.git
cd flask_app
python3 app.py
```

* Then, create the **Launch Template**.

### 2.2 Configuring Auto Scaling Group

1. Go to the **EC2 Dashboard** > **Auto Scaling Groups** section.
2. Click the **Create Auto Scaling Group** button.
3. Select the **Launch Template** you created.
4. Set Minimum: **1**, Maximum: **3**, Desired: **2** values.
5. Link the **Target Group** to the Load Balancer.

## 3. Final Checks

* Verify that your Flask application is running by entering the Load Balancer DNS name into your browser.
* Test the Auto Scaling Group's ability to scale EC2 instances based on traffic.

## Additional Resources

* [AWS Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)
* [Flask Deployment on AWS](https://flask.palletsprojects.com/en/2.0.x/deploying/)
* [GitHub Actions for CI/CD](https://docs.github.com/en/actions)
