# AWS Jenkins Instance

This repository provides instructions and configurations to set up a Jenkins server on an AWS EC2 instance.

## Prerequisites

Before proceeding, ensure you have the following:
- An **AWS account** with permissions to create EC2 instances
- **Terraform** (if using Infrastructure as Code)
- **AWS CLI** configured with the appropriate credentials
- **SSH key pair** for accessing the EC2 instance

## Setup Instructions

### 1. Launch an EC2 Instance
1. Navigate to the **AWS Management Console** and open the EC2 service.
2. Launch a new EC2 instance with the following specifications:
   - **AMI:** Ubuntu 20.04 or Amazon Linux 2
   - **Instance Type:** t2.medium or larger
   - **Security Group:** Allow inbound access on ports 22 (SSH), 8080 (Jenkins), and 50000 (JNLP agents)
   - **Key Pair:** Use an existing key pair or create a new one

### 2. Install Jenkins
SSH into the instance and run the following commands to install Jenkins:
```bash
sudo apt update && sudo apt install -y openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

### 3. Access Jenkins
Once installed, Jenkins runs on port **8080**. To access it:
1. Open a browser and go to `http://<EC2_PUBLIC_IP>:8080`
2. Retrieve the initial admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
3. Follow the setup wizard to complete the installation.

### 4. Configure AWS Credentials in Jenkins
To integrate AWS with Jenkins:
1. Install the **AWS Credentials** and **Pipeline: AWS Steps** plugins.
2. Go to **Manage Jenkins** → **Manage Credentials** → **Global Credentials**.
3. Add new credentials:
   - Type: AWS Credentials
   - ID: `aws-credentials`
   - Access Key & Secret Key: *Your AWS IAM User Keys*

### 5. (Optional) Automate Setup with Terraform
To provision the EC2 instance automatically, use the Terraform configuration provided in this repository:
```bash
terraform init
terraform apply -auto-approve
```

## Environment Variables
If required, configure the following environment variables in Jenkins:
```bash
AWS_ACCESS_KEY_ID=<your_aws_access_key>
AWS_SECRET_ACCESS_KEY=<your_aws_secret_key>
JENKINS_URL=http://<EC2_PUBLIC_IP>:8080
```

## Contributing
Feel free to open issues or submit pull requests to improve this setup.

## License
This project is licensed under the MIT License.
