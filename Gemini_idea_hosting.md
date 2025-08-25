# Complete Beginner's Guide to Deploying Your YaliAero MERN Application

This guide is designed for absolute beginners who have never used AWS or GitHub Actions before. We'll walk through every single step with detailed instructions to deploy your YaliAero MERN application.

## Table of Contents

1.  [Setting Up AWS Account](https://www.google.com/search?q=%23setting-up-aws-account)
2.  [Creating an EC2 Instance](https://www.google.com/search?q=%23creating-an-ec2-instance)
3.  [Connecting to Your EC2 Instance](https://www.google.com/search?q=%23connecting-to-your-ec2-instance)
4.  [Setting Up Your EC2 Instance](https://www.google.com/search?q=%23setting-up-your-ec2-instance)
5.  [Setting Up GitHub Repository](https://www.google.com/search?q=%23setting-up-github-repository)
6.  [Setting Up Dockerfiles and Docker Compose](https://www.google.com/search?q=%23setting-up-dockerfiles-and-docker-compose)
7.  [Setting Up GitHub Actions](https://www.google.com/search?q=%23setting-up-github-actions)
8.  [Deploying Your Application](https://www.google.com/search?q=%23deploying-your-application)
9.  [Troubleshooting](https://www.google.com/search?q=%23troubleshooting)
10. [Improving the Robustness of Your Deployment](https://www.google.com/search?q=%23improving-the-robustness-of-your-deployment)

-----

## Setting Up AWS Account

### Step 1: Create an AWS Account

1.  Open your web browser and go to [https://aws.amazon.com/](https://aws.amazon.com/)
2.  Click on the "Create an AWS Account" button in the top-right corner.
3.  Enter your email address and choose an AWS account name.
4.  Click "Continue".
5.  Choose "Personal" account type.
6.  Fill in your personal information.
7.  Enter your credit card information (AWS has a free tier, but they require a credit card).
8.  Complete the phone verification process.
9.  Choose the Basic Support plan (free).
10. Click "Complete Sign Up".

### Step 2: Sign in to AWS Console

1.  Go to [https://aws.amazon.com/](https://aws.amazon.com/).
2.  Click "Sign In to the Console" in the top-right corner.
3.  Enter your email address and password.
4.  You are now in the AWS Management Console.

-----

## Creating an EC2 Instance

An EC2 (Elastic Compute Cloud) instance is a virtual server in AWS's cloud. It's where your application will live and run.

### Step 1: Navigate to EC2 Service

1.  In the AWS Management Console, click on the search bar at the top.
2.  Type "EC2" and select "EC2" from the dropdown menu.
3.  You are now in the EC2 Dashboard.

### Step 2: Launch an EC2 Instance

1.  Click the orange "Launch instance" button.
2.  Enter a name for your instance (e.g., "YaliAero-MERN-Application").

### Step 3: Choose an Amazon Machine Image (AMI)

1.  Scroll down to the "Application and OS Images" section.
2.  Click on "Ubuntu" from the Quick Start options.
3.  Select "Ubuntu Server 22.04 LTS" or the latest LTS version available.

### Step 4: Choose an Instance Type

1.  Under "Instance type", select "t2.micro" (Free tier eligible).

### Step 5: Create a Key Pair

1.  Under "Key pair (login)", click the "Create new key pair" button.
2.  Enter a name for your key pair (e.g., "yaliaero-app-key").
3.  Keep the default settings (RSA, .pem).
4.  Click "Create key pair".
5.  The key file will automatically download to your computer.
      - **IMPORTANT**: Save this file in a secure location. You cannot download it again\!

### Step 6: Configure Network Settings

1.  Under "Network settings", click "Edit".
2.  Make sure "Auto-assign public IP" is set to "Enable".
3.  Under "Firewall (security groups)", select "Create security group".
4.  Make sure the following rules are added:
      - **SSH: Port 22, Source: Anywhere (0.0.0.0/0)** (For connecting to your instance)
      - **HTTP: Port 80, Source: Anywhere (0.0.0.0/0)** (For web traffic)
      - **HTTPS: Port 443, Source: Anywhere (0.0.0.0/0)** (For secure web traffic)

**Important Security Note**: Do not open ports `5000` (backend) or `27017` (database) in your security group. These services will run within a private Docker network and will not be exposed to the public internet. Nginx, running on ports 80 and 443, will be the only entry point to your application, which is a much more secure configuration.

### Step 7: Configure Storage

1.  Under "Configure storage", keep the default settings (8 GB gp2).

### Step 8: Launch the Instance

1.  Review all settings.
2.  Click the "Launch instance" button at the bottom right.

### Step 9: Note Your Instance Details

1.  In the EC2 instances dashboard, select your newly created instance.
2.  In the bottom panel, find and note down the **Public IPv4 address**.

-----

## Connecting to Your EC2 Instance

### For Windows Users

#### Step 1: Set Permissions on Your Key File

1.  Open Windows Terminal, Command Prompt, or PowerShell.
2.  Navigate to the directory where your `.pem` file is saved.
3.  Run the following command to ensure the key is not accessible to other users:
    ```bash
    icacls your-key-file.pem /inheritance:r /grant:r "%username%:(R)"
    ```

#### Step 2: Connect Using SSH

1.  In Windows Terminal, Command Prompt, or PowerShell, run:
    ```bash
    ssh -i your-key-file.pem ubuntu@your-public-ip
    ```
2.  Type "yes" if asked to confirm the connection.

### For Mac/Linux Users

#### Step 1: Set Permissions on Your Key File

1.  Open Terminal.
2.  Navigate to the directory where your `.pem` file is saved.
3.  Run the command:
    ```bash
    chmod 400 your-key-file.pem
    ```

#### Step 2: Connect Using SSH

1.  In Terminal, run:
    ```bash
    ssh -i your-key-file.pem ubuntu@your-public-ip
    ```
2.  Type "yes" if asked to confirm the connection.

-----

## Setting Up Your EC2 Instance

Once you're connected to your EC2 instance, run these commands to set up the environment.

```bash
# Update the system
sudo apt update
sudo apt upgrade -y

# Install Docker and Docker Compose plugin
sudo apt install -y docker-ce docker-compose-plugin

# Add the ubuntu user to the docker group
sudo usermod -aG docker ubuntu

# Log out and log back in for the group changes to take effect
exit
```

Reconnect to your instance using the same method as before, then continue:

```bash
# Verify Docker is working
docker --version

# Verify Docker Compose is working
docker compose version

# Install Git and Certbot
sudo apt install -y git certbot python3-certbot-nginx
```

-----

## Setting Up GitHub Repository

### Step 1: Create a GitHub Account (if you don't have one)

1.  Go to [https://github.com/](https://github.com/)
2.  Click "Sign up".

### Step 2: Create a New Repository

1.  Click the "+" icon in the top-right corner.
2.  Select "New repository".
3.  Enter a repository name (e.g., "YaliAero\_Website").
4.  Choose "Public" or "Private".
5.  Click "Create repository".

### Step 3: Push Your Code to GitHub

On your local machine (not the EC2 instance), navigate to your project folder and run:

```bash
git init
git add .
git commit -m "Initial commit for YaliAero MERN app"
git remote add origin https://github.com/your-username/YaliAero_Website.git
git push -u origin main
```

Replace "your-username" with your actual GitHub username.

-----

## Setting Up Dockerfiles and Docker Compose

You will use the same Dockerfiles and Docker Compose file provided in the original guide. No changes are needed here.

-----

## Setting Up GitHub Actions

### Step 1: Set Up Docker Hub

1.  Go to [https://hub.docker.com/](https://hub.docker.com/)
2.  Click "Sign Up" if you don't have an account.
3.  Create two repositories: `yaliaero-backend` and `yaliaero-frontend`.

### Step 2: Add GitHub Secrets

In your GitHub repository, add these secrets.

1.  Name: `EC2_HOST` (Value: Your EC2 instance's public IPv4 address)
2.  Name: `EC2_USER` (Value: `ubuntu`)
3.  Name: `SSH_PRIVATE_KEY` (Value: The full content of your `.pem` file from your local machine.)
4.  Name: `DOCKER_USERNAME` (Value: Your Docker Hub username)
5.  Name: `DOCKER_PASSWORD` (Value: Your Docker Hub password or access token)
6.  Name: `MONGO_PASSWORD` (Value: A strong, secure password for MongoDB)
7.  Name: `JWT_SECRET` (Value: A secure random string)
8.  Name: `AWS_ACCESS_KEY_ID` (Value: Your AWS Access Key ID)
9.  Name: `AWS_SECRET_ACCESS_KEY` (Value: Your AWS Secret Access Key)
10. Name: `AWS_REGION` (Value: `ap-south-1`)
11. Name: `S3_BUCKET` (Value: The name of your S3 bucket)
12. Name: `VERIFICATION_TOKEN_SECRET` (Value: A secure random string)
13. Name: `ASSOCIATE_VERIFICATION_TOKEN_SECRET` (Value: A secure random string)
14. Name: `EMAIL_HOST` (Value: `smtp.gmail.com`)
15. Name: `EMAIL_PORT` (Value: `587`)
16. Name: `EMAIL_USER` (Value: Your email address for sending emails)
17. Name: `EMAIL_PASS` (Value: Your email password or app-specific password)
18. Name: `FRONTEND_BASE_URL` (Value: The public URL of your deployed frontend, e.g., `https://yourdomain.com`)
19. Name: `NODE_ENV` (Value: `production`)

### Step 3: Updated GitHub Actions Workflow

This workflow uses a GitHub-managed runner for building and pushing images and a separate job for deployment via SSH.

```yaml
name: Deploy YaliAero MERN Application

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Source
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Backend Image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/yaliaero-backend:latest ./server

      - name: Build Frontend Image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/yaliaero-frontend:latest ./client-vite

      - name: Push Images to Docker Hub
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/yaliaero-backend:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/yaliaero-frontend:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2 via SSH
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /home/ubuntu/YaliAero_Website
            docker compose down || true
            docker pull ${{ secrets.DOCKER_USERNAME }}/yaliaero-backend:latest
            docker pull ${{ secrets.DOCKER_USERNAME }}/yaliaero-frontend:latest
            docker compose up -d
```

-----

## Deploying Your Application

### Step 1: Initial Setup on EC2

This step is done **manually, just once**, after you've SSHed into your EC2 instance.

```bash
# Clone your repository
git clone https://github.com/your-username/YaliAero_Website.git
cd YaliAero_Website

# Create a local .env file for Docker Compose to read
nano .env
```

Inside the `nano` editor, paste all the environment variables from your GitHub secrets in `KEY=VALUE` format. Save and exit the editor.

### Step 2: Configure Nginx as a Reverse Proxy

On your EC2 instance, create an Nginx configuration file to handle traffic and redirect it to the correct Docker containers.

```bash
# Create the Nginx config file
sudo nano /etc/nginx/sites-available/yaliaero.conf
```

Paste this configuration, replacing `yourdomain.com` with your actual domain:

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Save and exit. Then, enable the site and test the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/yaliaero.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Step 3: Configure Domain and SSL

1.  **Configure Your Domain**:

      - The first step is to point your domain name to your EC2 instance's public IP address.

    ### Using AWS Route 53 for Domain Management and Registration

    AWS Route 53 is a highly available and scalable Domain Name System (DNS) web service that also allows you to register new domains.

    1.  **Register a New Domain in Route 53 (if you don't have one)**:

          * Navigate to the Route 53 service in the AWS Management Console.
          * In the left sidebar, click on "Registered domains" and then "Register domain".
          * Enter the domain name you want (e.g., `yourdomain.com`) and click "Check".
          * If the domain is available, follow the prompts to complete the registration process.

    2.  **Create a Hosted Zone in Route 53**:

          * If you just registered a domain, a hosted zone will be automatically created for you.
          * If you are using a domain purchased from another registrar, you will need to create a hosted zone manually. Navigate to "Hosted zones" in the left sidebar, click "Create hosted zone", enter your domain name, select "Public hosted zone", and click "Create hosted zone".
          * **Note**: If you created a new hosted zone for a domain from another registrar, Route 53 will provide you with a set of Name Server (NS) records. You must update your domain registrar's settings with these new Name Servers to delegate DNS management to Route 53.

    3.  **Create an 'A' Record in Route 53**:

          * In your Route 53 hosted zone, click "Create record".
          * For "Record name", leave it blank if you are configuring the root domain (e.g., `yourdomain.com`), or enter a subdomain (e.g., `www`).
          * For "Record type", select **"A - Routes traffic to an IPv4 address and some AWS resources"**.
          * For "Value", enter your EC2 instance's Public IPv4 address.
          * Click "Create records".
          * **Note**: DNS changes can take some time to propagate across the internet (up to 48 hours).

2.  **Set Up SSL Certificate**:

    ```bash
    # Get SSL certificate for your domain
    sudo certbot --nginx -d yourdomain.com
    # Follow the prompts to configure HTTPS
    ```

3.  **Verify SSL Configuration**:

    ```bash
    # Test SSL renewal
    sudo certbot renew --dry-run
    ```

-----

## Troubleshooting

### Issue: Application Not Accessible

1.  Check that containers are running: `docker ps`
2.  Check Nginx status: `sudo systemctl status nginx`
3.  Verify that your EC2 security group allows traffic on ports 80 and 443.

### Issue: Docker Commands Not Working

1.  Make sure Docker is installed and running: `sudo systemctl status docker`
2.  Make sure your user is in the docker group: `groups`. If not, run `sudo usermod -aG docker $USER` and log out/in.

### Issue: GitHub Actions Deployment Failure

1.  Go to your GitHub repository's **Actions** tab.
2.  Click on the failed workflow run to view the logs.

-----

## Improving the Robustness of Your Deployment

This guide provides a solid foundation, but to make your deployment more robust, secure, and scalable, consider these additional improvements:

  * **Infrastructure as Code (IaC)**: Use tools like **Terraform** or **CloudFormation** to automate the creation of your AWS infrastructure.
  * **Centralized Logging and Monitoring**: Implement a centralized logging system (e.g., the **ELK Stack** or **Prometheus**) to aggregate and monitor logs from all your containers in one place.
  * **Scalability**: For increased traffic, consider moving to a container orchestration platform like **Kubernetes** or **AWS ECS (Elastic Container Service)**, which can automate scaling and load balancing.
  * **Enhanced Security**: Instead of a self-hosted runner, use a more secure approach for SSH key management, such as an SSH agent forwarding or a dedicated vault service.
  * **Zero-Downtime Deployments**: Implement a rollback strategy to avoid downtime during deployments.