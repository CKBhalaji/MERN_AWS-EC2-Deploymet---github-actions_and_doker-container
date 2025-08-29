## Deployment Steps Summary

To ensure a smooth deployment, follow these steps in the given order:

1.  **Set up an AWS Account**: Create an AWS account and sign in to the AWS Console.
2.  **Configure AWS SES for Email**: Verify your email address, get SMTP credentials, request production access, and configure your application with the credentials.
3.  **Create an EC2 Instance**: Launch an EC2 instance, choose an Ubuntu AMI, select an instance type, create a key pair, configure network settings, configure storage, and note your instance details.
4.  **Connect to Your EC2 Instance**: Connect to your EC2 instance using SSH (different instructions for Windows and Mac/Linux users).
5.  **Set Up Your EC2 Instance**: Update the system, install Docker and Docker Compose, start Docker, verify Docker is working, install Git, and install Certbot for SSL.
6.  **Nginx Configuration**: Configure Nginx on your EC2 instance using the `yaliaero.conf` file. After deployment, replace `localhost` with your actual domain name and ensure the SSL certificates are located in the specified paths.
7.  **Set Up GitHub Repository**: Create a GitHub account (if you don't have one), create a new repository, and push your code to GitHub.
8.  **Set Up Dockerfiles and Docker Compose**: Create `server/Dockerfile`, `client-vite/Dockerfile`, `.dockerignore` files, and `docker-compose.yml`.
9.  **Set up the Amazon DocumentDB (with MongoDB compatibility) Database**: Create a DocumentDB Cluster, Configure VPC Security Groups, Obtain the Connection String and Environment Variables, Import the TLS Certificate.
10. **Create and Configuring an ECR Repository**: Create two separate repositories (backend and frontend) and retrieve the repository URIs.
11. **Create and Configuring an IAM User**: Create an IAM user with the `AmazonEC2ContainerRegistryPowerUser`, `AmazonEC2ContainerRegistryFullAccess`, `AmazonEC2FullAccess` and `AmazonS3FullAccess` policies and get the access keys for the IAM user. Also, get your AWS Account ID and ECR Repository Name.
12. **Setting Up GitHub Actions**: Set up GitHub Actions Runner on EC2 and add GitHub Secrets.
13. **Deploying Your Application**: Set Up Docker Network, Configure GitHub Actions Runner, Understanding Container Management, Configure Domain and SSL, and Verify Deployment.

---

# Complete Beginner's Guide to Deploying Your YaliAero MERN Application in AWS EC2 with AWS ECR

This guide is designed for absolute beginners who have never used AWS or GitHub Actions before. We'll walk through every single step with detailed instructions to deploy your YaliAero MERN application.

## Table of Contents

1. [Setting Up AWS Account](#setting-up-aws-account)
2. [Creating an EC2 Instance](#creating-an-ec2-instance)
3. [Connecting to Your EC2 Instance](#connecting-to-your-ec2-instance)
4. [Setting Up Your EC2 Instance](#setting-up-your-ec2-instance)
5. [Nginx Configuration](#nginx-configuration)
6. [Setting Up GitHub Repository](#setting-up-github-repository)
7. [Setting Up Dockerfiles and Docker Compose](#setting-up-dockerfiles-and-docker-compose)
8. [Creating and Configuring an ECR Repository](#creating-and-configuring-an-ecr-repository)
9. [Creating and Configuring an IAM User](#creating-and-configuring-an-iam-user)
10. [Setting Up GitHub Actions](#setting-up-github-actions)
11. [Deploying Your Application](#deploying-your-application)
12. [Troubleshooting](#troubleshooting)

---

### Introduction to Key Concepts

- **Docker**: Docker is a platform that uses OS-level virtualization to deliver software in packages called **containers**. A container is a lightweight, standalone, and executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.
- **Continuous Integration/Continuous Deployment (CI/CD)**: This is a software development practice that automates the process of building, testing, and deploying code changes.
- **Amazon ECR**: ECR is a fully managed Docker container registry offered by AWS. It allows you to easily store, manage, and deploy your Docker container images. It integrates seamlessly with other AWS services and is a secure place to store your images.
- **GitHub Actions**: GitHub Actions is a CI/CD platform that lets you automate your build, test, and deployment pipeline directly from your GitHub repository.

---

### Setting Up AWS Account

### Step 1: Create an AWS Account

1. Open your web browser and go to [https://aws.amazon.com/](https://aws.amazon.com/)
2. Click on the "Create an AWS Account" button in the top-right corner
3. Enter your email address and choose an AWS account name
4. Click "Continue"
5. Choose "Personal" account type
6. Fill in your personal information
7. Enter your credit card information (AWS has a free tier, but they require a credit card)
8. Complete the phone verification process
9. Choose the Basic Support plan (free)
10. Click "Complete Sign Up"

### Step 2: Sign in to AWS Console

1. Go to [https://aws.amazon.com/](https://aws.amazon.com/)
2. Click "Sign In to the Console" in the top-right corner
3. Enter your email address and password
4. You are now in the AWS Management Console

---

## Using AWS SES for Email

Using AWS for email is a multi-step process that ensures your application is secure and your emails have high deliverability. Here is a complete, step-by-step guide to get everything configured in one go.

### Step 1: Verify Your Email Address 📧

1. This is the fastest and easiest way to get started.
2. Log in to the AWS Management Console and navigate to the SES Dashboard.
3. In the left menu, under Configuration, click Verified identities.
4. Click Create identity. For Identity type, choose Email address.
5. Enter the exact email address you want to use (e.g., info@yourdomain.com).
6. AWS will send a verification email to that address. You must open the email and click the link to complete the verification. The status in your dashboard will change to "Verified" when done.

### Step 2: Get Your SMTP Credentials 🔑

1. Once your email is verified, you need to generate a specific set of credentials for your application. These are different from your AWS console login or IAM user keys.
2. In the SES Dashboard, click SMTP Settings in the left menu.
3. Click Create My SMTP Credentials. This will take you to the IAM console.
4. An IAM user will be created. You can leave the name as the default or change it.
5. After the user is created, AWS will display your unique SMTP username and SMTP password. Copy and save these credentials immediately! You will not be able to view the password again.

### Step 3: Request Production Access 🚀

1. New AWS accounts are in a sandbox environment, which means you can only send emails to verified addresses. For a production application, you must request to move out of the sandbox.
2. In the SES Dashboard, go to your Account dashboard.
3. Click Request production access and fill out the form. You will need to explain what your application does and the types of emails you'll be sending.
4. Submit the request. AWS typically reviews these within 24-48 hours. Once approved, you can send emails to any recipient.

### Step 4: Configure Your Application ⚙️

Now, you can use the credentials and host information you've gathered to fill in your application's environment variables.

```bash
EMAIL_HOST = email-smtp.[your-region].amazonaws.com (e.g., email-smtp.us-east-1.amazonaws.com)

EMAIL_PORT = 587

EMAIL_USER = Your unique SMTP username from Step 2.

EMAIL_PASS = Your unique SMTP password from Step 2.
```

---

### Creating an EC2 Instance

### Step 1: Navigate to EC2 Service

1. In the AWS Management Console, click on the search bar at the top
2. Type "EC2" and select "EC2" from the dropdown menu
3. You are now in the EC2 Dashboard

### Step 2: Launch an EC2 Instance

1. Click the orange "Launch instance" button
2. Enter a name for your instance (e.g., "YaliAero-MERN-Application")

### Step 3: Choose an Amazon Machine Image (AMI)

1. Scroll down to the "Application and OS Images" section
2. Click on "Ubuntu" from the Quick Start options
3. Select "Ubuntu Server 22.04 LTS" or the latest LTS version available

### Step 4: Choose an Instance Type

1. Under "Instance type", select "t2.micro" (Free tier eligible)
   - Note: If you plan to run MongoDB on the same instance, consider selecting "t2.small" or larger for better performance.

### Step 5: Create a Key Pair

1. Under "Key pair (login)", click the "Create new key pair" button
2. Enter a name for your key pair (e.g., "yaliaero-app-key")
3. Keep the default settings (RSA, .pem)
4. Click "Create key pair"
5. The key file will automatically download to your computer
   - **IMPORTANT**: Save this file in a secure location. You cannot download it again!

### Step 6: Configure Network Settings

1. Under "Network settings", click "Edit"
2. Make sure "Auto-assign public IP" is set to "Enable"
3. Under "Firewall (security groups)", select "Create security group"
4. Make sure the following rules are added:
   - SSH: Port 22, Source: Anywhere (0.0.0.0/0) (For connecting to your instance)
   - HTTP: Port 80, Source: Anywhere (0.0.0.0/0) (For web traffic)
   - HTTPS: Port 443, Source: Anywhere (0.0.0.0/0) (For secure web traffic)

**Important Security Note**: You do not need to open ports `5000` (backend) or `27017` (database) in your security group. These services will run within a private Docker network and will not be exposed to the public internet. Nginx, running on ports 80 and 443, will be the only entry point to your application, which is a much more secure configuration.

### Step 7: Configure Storage

1. Under "Configure storage", keep the default settings (8 GB gp2)
   - You can increase this if you expect to store a lot of data

### Step 8: Launch the Instance

1. Review all settings
2. Click the "Launch instance" button at the bottom right
3. You'll see a success message. Click "View all instances" to go to your EC2 instances dashboard

### Step 9: Note Your Instance Details

1. In the EC2 instances dashboard, select your newly created instance
2. In the bottom panel, find and note down:
   - Public IPv4 address (e.g., 12.34.56.78)
   - Public IPv4 DNS (e.g., ec2-12-34-56-78.compute-1.amazonaws.com)

---

### Connecting to Your EC2 Instance

### For Windows Users

#### Step 1: Set Permissions on Your Key File

1. Open Windows Terminal, Command Prompt, or PowerShell
2. Navigate to the directory where your .pem file is saved
3. Run the following command to ensure the key is not accessible to other users:

```

icacls your-key-file.pem /inheritance:r /grant:r "%username%:(R)"

```

Replace "your-key-file.pem" with your actual key file name

#### Step 2: Connect Using SSH

1. In Windows Terminal, Command Prompt, or PowerShell, run:

```

ssh -i your-key-file.pem ubuntu@your-public-ip

```

- Replace "your-key-file.pem" with your actual key file name
- Replace "your-public-ip" with your EC2 instance's public IP address

2. Type "yes" if asked to confirm the connection
3. You should now be connected to your EC2 instance

### For Mac/Linux Users

#### Step 1: Set Permissions on Your Key File

1. Open Terminal
2. Navigate to the directory where your .pem file is saved
3. Run the command:

```

chmod 400 your-key-file.pem

```

Replace "your-key-file.pem" with your actual key file name

#### Step 2: Connect Using SSH

1. In Terminal, run:

```

ssh -i your-key-file.pem ubuntu@your-public-ip

```

- Replace "your-key-file.pem" with your actual key file name
- Replace "your-public-ip" with your EC2 instance's public IP address

2. Type "yes" if asked to confirm the connection

---

### Setting Up Your EC2 Instance

Once you're connected to your EC2 instance, run these commands to set up the environment:

```bash
# Update the system
sudo apt update
sudo apt upgrade -y

# Install Docker and Docker Compose
sudo apt install -y docker.io docker-compose-plugin

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker
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

# Install Git
sudo apt install -y git

# Install Certbot for SSL
sudo apt install -y certbot python3-certbot-nginx
```

---

### Nginx Configuration

After setting up your EC2 instance, you need to configure Nginx to serve your application. There are two Nginx configurations used in this project:

1.  **`client-vite/frontend-nginx.conf`**: This configuration is copied into the frontend Docker image. It's used for serving the static frontend files and proxying API requests to the backend container _within the Docker network_.
2.  **`yaliaero.conf`**: This configuration is intended for the Nginx server running directly on the EC2 instance (the host). It acts as a main reverse proxy, handling SSL and routing traffic to the Docker containers.

**For the EC2 instance deployment:**

You will use the `yaliaero.conf` file. Follow these steps to configure Nginx on your EC2 instance:

1.  Connect to your EC2 instance using SSH.
2.  Use a text editor to create a new file in the `/etc/nginx/conf.d/` directory. We recommend using `vim`, which is a powerful text editor commonly found on Linux systems. To use `vim`, follow these steps:
    a. Open a new file in `vim` with the following command: `sudo vim /etc/nginx/conf.d/yaliaero.conf`
    b. Press the `i` key to enter insert mode. This will allow you to type text into the file.
    c. Copy the Nginx configuration from this guide and paste it into the file.
    d. Press the `Esc` key to exit insert mode.
    e. Type `:wq` and press `Enter` to save the file and exit `vim`.
    If you prefer to use `nano`, you can use the following command: `sudo nano /etc/nginx/conf.d/yaliaero.conf`. Nano is a simpler text editor that is easier to use for beginners.
3.  Copy the Nginx configuration from this guide and paste it into the file.
4.  Save the file and exit the text editor.
5.  Test the Nginx configuration: `sudo nginx -t`
6.  If the configuration is correct, reload Nginx: `sudo systemctl reload nginx`

### How the Nginx Configuration Works

Nginx acts as a reverse proxy in this setup, handling incoming requests and forwarding them to the appropriate backend servers. Here's a breakdown of how the configuration works:

- **server blocks**: The `server` blocks define the configuration for different domains or subdomains. In this case, we have two server blocks: one for HTTP (port 80) and one for HTTPS (port 443). The HTTP server block redirects all traffic to HTTPS.
- **location blocks**: The `location` blocks define how Nginx handles requests to different paths.
  - The `location /` block handles all requests to the root path and forwards them to the frontend container (`frontend`) on port 80.
  - The `location /api/` block handles all requests to paths starting with `/api/` and forwards them to the backend container.
- **proxy_pass directive**: The `proxy_pass http://backend:5000;` directive specifies the address of the backend server to which the request should be forwarded. The line tells Nginx to send the request to the backend container at its internal address on port 5000. This communication happens securely inside the private Docker network and does **not** expose port 5000 to the public internet.
- **proxy_set_header directives**: The `proxy_set_header` directives pass information about the original request to the backend servers. This information includes the host, the client's IP address, and the protocol used (HTTP or HTTPS).
- **add_header Cache-Control directive**: The `add_header Cache-Control` directive sets the cache control headers for static assets served by the frontend. This tells the browser how long to cache these assets, which can improve performance.
- **error_page directive**: The `error_page` directive handles errors. In this case, it specifies that if an error occurs (500, 502, 503, or 504), Nginx should display the `50x.html` file.

This Nginx configuration is correctly aligned with the project structure because it forwards requests to the appropriate containers based on the URL path. Requests to the root path are handled by the frontend container, and requests to the `/api/` path are handled by the backend container.

```nginx
server {
   listen 80;
   server_name localhost;
   return 301 https://$host$request_uri;
}

server {
   listen 443 ssl;
   server_name localhost;

   ssl_certificate /etc/letsencrypt/live/localhost/fullchain.pem;
   ssl_certificate_key /etc/letsencrypt/live/localhost/privkey.pem;

   location / {
      proxy_pass http://frontend:80;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      add_header Cache-Control "public, max-age=3600";
   }

   location /api/ {
      proxy_pass http://backend:5000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
   }

   error_page 500 502 503 504 /50x.html;
   location = /50x.html {
      root /usr/share/nginx/html;
   }
}
```

**Important**: After deployment, replace `localhost` with your actual domain name. Also, make sure that the SSL certificates are located in the specified paths (`/etc/letsencrypt/live/yourdomain.com/fullchain.pem` and `/etc/letsencrypt/live/yourdomain.com/privkey.pem`).

After deploying your application and setting up your custom domain, you need to update the Nginx configuration file (`yaliaero.conf`) with your domain name and ensure the SSL certificates are correctly configured. Follow these steps:

1.  Connect to your EC2 instance using SSH.
2.  Open the Nginx configuration file (`/etc/nginx/conf.d/yaliaero.conf`) using a text editor (e.g., `vim` or `nano`): `sudo vim /etc/nginx/conf.d/yaliaero.conf`
3.  Update the `server_name` directives in both the H`TTP and HTTPS server blocks to use your custom domain name. For example:

    ```nginx
    server {
        listen 80;
        server_name yourdomain.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name yourdomain.com;
    ```

4.  Verify that the `ssl_certificate` and `ssl_certificate_key` directives in the HTTPS server block point to the correct paths for your SSL certificates. Replace `yourdomain.com` with your actual domain name:
    ```nginx
    ssl_certificate /etc/letsencrypt/live/[yourdomain.com/fullchain.pem](https://yourdomain.com/fullchain.pem);
    ssl_certificate_key /etc/letsencrypt/live/[yourdomain.com/privkey.pem](https://yourdomain.com/privkey.pem);
    ```
5.  Save the file and exit the text editor.
6.  Test the Nginx configuration: `sudo nginx -t`
7.  If the configuration is correct, reload Nginx: `sudo systemctl reload nginx`

---

### Setting Up GitHub Repository

### Step 1: Create a GitHub Account (if you don't have one)

1.  Go to [https://github.com/](https://github.com/)
2.  Click "Sign up"
3.  Follow the instructions to create your account

### Step 2: Create a New Repository

1.  Click the "+" icon in the top-right corner
2.  Select "New repository"
3.  Enter a repository name (e.g., "YaliAero_Website")
4.  Choose "Public" or "Private"
5.  Click "Create repository"

### Step 3: Push Your Code to GitHub

On your local machine (not the EC2 instance), navigate to your project folder (the root of `YaliAero_Website`) and run:

```bash
# Initialize Git repository (if not already done)
git init

# Add all files
git add .

# Commit the files
git commit -m "Initial commit for YaliAero MERN app"

# Add the remote repository
git remote add origin [https://github.com/your-username/YaliAero_Website.git](https://github.com/your-username/YaliAero_Website.git)

# Push to GitHub
git push -u origin main
```

Replace "your-username" with your actual GitHub username.

---

### Setting Up Dockerfiles and Docker Compose

We will containerize your frontend and backend applications using Docker. The deployment process utilizes a multi-container architecture managed by `docker-compose.yml`. The `deploy.yml` file uses `docker-compose` to manage the deployment.

### Step 1: Create `server/Dockerfile`

This Dockerfile builds the Node.js backend application using a multi-stage build process for optimized image size and includes Python dependencies.

Create a file named `Dockerfile` inside the `server/` directory with the following content:

```dockerfile
FROM node:20-alpine

# Set the working directory in the container
WORKDIR /app

# Install Python and pip, and build dependencies
# Use a single RUN command to minimize layers and clean up apk cache
RUN apk add --no-cache python3 py3-pip gcc musl-dev linux-headers fftw-dev build-base python3-dev && \
    python3 -m venv /opt/venv && \
    /opt/venv/bin/pip install --no-cache-dir --upgrade pip

# Copy Python requirements and install them
COPY requirements.txt ./
RUN /opt/venv/bin/pip install --no-cache-dir --verbose -r requirements.txt

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install Node.js dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 5000

# Ensure Python from venv is available in the PATH for any Python scripts called by Node.js
ENV PATH="/opt/venv/bin:$PATH"

# Define the command to run the application
CMD ["npm", "start"]
```

### Step 2: Create `client-vite/Dockerfile`

This Dockerfile builds the React frontend application and serves it using Nginx.

Create a file named `Dockerfile` inside the `client-vite/` directory with the following content:

```dockerfile
FROM node:20-alpine AS build

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the React application
# ENV NODE_ENV=development
RUN VITE_API_BASE_URL=/api npm run build --force

# Use Nginx to serve the static files
FROM nginx:alpine

# Copy the built React app to Nginx's public directory
COPY --from=build /app/dist /usr/share/nginx/html

# Copy the custom Nginx configuration
COPY frontend-nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Step 3: Create `.dockerignore` files

To optimize Docker builds, create `.dockerignore` files in both the `server/` and `client-vite/` directories to prevent unnecessary files from being copied into the Docker images.

**`server/.dockerignore`**:

```
node_modules
```

**`client-vite/.dockerignore`**:

```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
dist
```

### Step 4: Create docker-compose.yml

This file defines and runs the multi-container Docker application. It uses environment variable substitution for sensitive information, which should be provided at runtime from the environment where `docker-compose` is executed (e.g., from GitHub Actions secrets or a local .env file).

Create a file named `docker-compose.yml` in the root of your project directory (`YaliAero_Website/`).

```yaml
version: '3.8'
services:
  backend:
    image: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}_backend:latest
    container_name: yaliaero-backend-container
    restart: always
    environment:
      NODE_ENV: ${NODE_ENV}
      MONGO_URI: ${MONGO_URI}
      JWT_SECRET: ${JWT_SECRET}
      PORT: ${PORT}
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      AWS_REGION: ${AWS_REGION}
      S3_BUCKET: ${S3_BUCKET}
      VERIFICATION_TOKEN_SECRET: ${VERIFICATION_TOKEN_SECRET}
      ASSOCIATE_VERIFICATION_TOKEN_SECRET: ${ASSOCIATE_VERIFICATION_TOKEN_SECRET}
      EMAIL_HOST: ${EMAIL_HOST}
      EMAIL_PORT: ${EMAIL_PORT}
      EMAIL_USER: ${EMAIL_USER}
      EMAIL_PASS: ${EMAIL_PASS}
      FRONTEND_BASE_URL: ${FRONTEND_BASE_URL}
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD}
    networks:
      - mern-network

  frontend:
    image: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}_frontend:latest
    container_name: yaliaero-frontend-container
    restart: always
    ports:
      - "80:80"
    environment:
      VITE_API_BASE_URL: http://backend:5000
    depends_on:
      - backend
    networks:
      - mern-network

networks:
  mern-network:
    driver: bridge
```

The `docker-compose.yml` has been updated to remove the `mongodb` service and the volume, as you will be connecting to a DocumentDB instance running outside of this Docker setup. The `MONGO_URI` variable is now a general environment variable.

## Setting up the Amazon DocumentDB (with MongoDB compatibility) Database

Since you are not using a local MongoDB instance, you will need to set up a managed DocumentDB cluster on AWS. This section provides a complete guide for that process.

### Step 1: Create a DocumentDB Cluster 📄

1. Navigate to the AWS Management Console and search for "DocumentDB".
2. In the left navigation pane, select Clusters.
3. Click the orange Create button.
4. Under Configuration, choose the MongoDB-compatible version you need (e.g., 5.0.0).
5. Provide a Cluster identifier (e.g., yaliaero-database).
6. Under Authentication, create a master username and password. Note these down securely, as you will need them for your MONGO_URI.
7. Under Cluster size, select the appropriate instance class for your needs (e.g., t3.medium).
8. Under Connectivity, choose the VPC and subnet group where your EC2 instance is located. This is crucial for allowing your application to connect to the database.
9. Under VPC security groups, select the security group you created for your EC2 instance. This will grant your application access to the DocumentDB cluster.
10. Under Cluster options, make sure to set the Port to 27017 to maintain compatibility with MongoDB drivers.
11. Click Create cluster. It may take a few minutes for the cluster to be provisioned.

### Step 2: Configure VPC Security Groups 🛡️

1. You must allow your EC2 instance to communicate with the DocumentDB cluster.
2. In the AWS Management Console, search for "VPC" and go to the VPC Dashboard.
3. In the left navigation pane, select Security groups.
4. Find the security group you used for your DocumentDB cluster (it's often named after the cluster, like docdb-security-group-xxxxxxxx).
5. Click on the security group ID, then select the Inbound rules tab.
6. Click Edit inbound rules.
7. Add a new rule:
   Type: Custom TCP
   Port range: 27017
   Source: Select the security group of your EC2 instance. This ensures that only your application can access the database, not the public internet.
8. Click Save rules.

### Step 3: Obtain the Connection String and Environment Variables 🔗

1. After your DocumentDB cluster is available, you need to get the connection string to use in your application.
2. In the DocumentDB console, go to Clusters.
3. Click on your cluster identifier.
4. In the Connectivity & security tab, find the Connection strings section.
5. Copy the connection string. It will look something like this:


    ``` bash
      mongodb://<user>:<password>@<cluster_endpoint>:27017/?tls=true&replicaSet=rs0&readPreference=secondaryPreferred&retryWrites=false
    ```

6. Replace <user> and <password> with the credentials you created in Step 1.
7. Note that the default connection string requires TLS. You'll need to update your application code to handle this. Most MongoDB drivers have a tls=true or similar option.
8. Update your docker-compose.yml environment variable configuration and your GitHub Actions secrets with the new MONGO_URI. Your docker-compose.yml will look like this:

```YAML
version: '3.8'
services:
  backend:
    image: ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}-backend:latest
    container_name: yaliaero-backend-container
    restart: always
    environment:
      NODE_ENV: ${NODE_ENV}
      MONGO_URI: ${{ MONGO_URI }}
      ...
```

9. You will store the full connection string, including the username and password, as a secret in GitHub Actions named MONGO_URI. For example, MONGO_URI=mongodb://[username]:[password]@[cluster_endpoint]:27017/?tls=true&replicaSet=rs0.

### Step 4: Import the TLS Certificate

1. DocumentDB clusters created after January 14, 2020 require you to use a TLS certificate to connect. You must download the Amazon DocumentDB TLS certificate to your EC2 instance.
2. Connect to your EC2 instance via SSH.
3. Run the following commands to download the certificate and install the necessary dependencies:

```bash
  # Install the TLS certificate (required for newer DocumentDB clusters)
  wget https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem
  mv rds-combined-ca-bundle.pem rds-ca.pem
```

4. You will need to update your application's database connection code to use this certificate. For a Node.js application using Mongoose, this will look something like this:

```javaScript
  const mongoose = require('mongoose');

  // The `tlsCAFile` option points to the downloaded certificate file.
  mongoose
  .connect(process.env.MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    tls: true,
    tlsCAFile: 'rds-ca.pem'
  })
  .then(async () => {
    console.log('Connected to MongoDB');
```

---

### Creating and Configuring an ECR Repository

ECR is where you'll store your Docker images. This process can be done via the AWS Management Console or the AWS CLI.

### Step 1: Create the Repository

You will create two separate repositories: one for the backend image and one for the frontend image.

- **Using the AWS Management Console**:
  1.  Log in to your AWS account.
  2.  Navigate to the **ECR** service.
  3.  Click on **Create repository**.
  4.  Choose **Private** as the visibility setting.
  5.  Give your repository a unique and descriptive name (e.g., `yaliaero-backend`).
  6.  Click **Create repository**.
  7.  Repeat the process to create another repository named `yaliaero-frontend`.
- **Using the AWS CLI**:
  ```bash
  aws ecr create-repository --repository-name yaliaero-backend --region <your-aws-region>
  aws ecr create-repository --repository-name yaliaero-frontend --region <your-aws-region>
  ```

### Step 2: Retrieve Repository URI

After creation, note down the repository URI for both repositories. It will be in the format `<aws_account_id>.dkr.ecr.<your-aws-region>.amazonaws.com/yaliaero-backend`. You will use this URI in your GitHub Actions workflow.

---

### Creating and Configuring an IAM User

You need an IAM (Identity and Access Management) user with specific permissions to allow GitHub Actions to securely interact with your AWS account. This user will be granted the necessary access to push and pull images from ECR.

### Step 1: Create an IAM User (or) append to the alredy using IAM

1.  In the AWS Management Console, use the search bar to find and navigate to the **IAM** service.
2.  In the left navigation pane, click on **Users**, then click **Add users**.
3.  Enter a user name (e.g., `github-actions-user`).
4.  Check the box next to **"Provide user access to the AWS Management Console"** and select **"I want to create an IAM user"**. Then choose **"Password"**.
5.  Click **"Next"**.
6.  On the **"Set permissions"** page, select **"Attach policies directly"**.
7.  In the search bar, type `ECR`.
8.  Select the check box next to **`AmazonEC2ContainerRegistryPowerUser`**. This managed policy provides the necessary permissions for building and deploying images to ECR.
9.  Click **"Next"** and then **"Create user"**.

(or)

1.  **Navigate to IAM**: Go to the AWS Management Console, search for **IAM**, and select it.
2.  **Find the User**: In the left-hand navigation pane, click on Users, and then click on the IAM user you want to modify.
3.  **Attach a New Policy**: On the user's details page, navigate to the **Permissions** tab and click on **Add permissions**.
4.  **Attach Policies Directly**: Choose **Attach policies directly**.
5.  **Search and Attach**: In the search bar, type `ECR`. Select the checkbox next to the managed policy named **`AmazonEC2ContainerRegistryPowerUser`**, **`AmazonEC2ContainerRegistryFullAccess`**, **`AmazonEC2FullAccess`** **`AmazonS3FullAccess`**.
6.  **Review and Save**: Click **Next**, and then **Add permissions** to save the changes.
    Once you have completed these steps, your IAM user will have the necessary permissions to interact with both S3 and ECR.

### Step 2: Get the Access Keys for the IAM User

1.  After creating the user, you'll see a success page. Click on the user name you just created (e.g., `github-actions-user`).
2.  Navigate to the **"Security credentials"** tab.
3.  Under the **"Access keys"** section, click **"Create access key"**.
4.  Select **"Third-party service"** as the use case. Check the box to confirm, and click **"Next"**.
5.  On the next page, you will see your **Access key ID** and **Secret access key**.
    - **IMPORTANT**: Copy these two values immediately. You will **not** be able to see the secret access key again after you close this page. Save them in a secure place. These are the values for `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` that you'll add to GitHub Secrets.

### Step 3: Get Your AWS Account ID and ECR Repository Name

1.  **AWS Account ID**: You can find your 12-digit account ID in the top-right corner of the AWS Management Console. Click on your account name or ID to see the full account information. This is the value for `AWS_ACCOUNT_ID`.
2.  **ECR Repository Name**: The `ECR_REPOSITORY` secret in your GitHub repository will be the **base name** of your ECR repositories. In our case, this will be `yaliaero`. The GitHub Actions workflow will automatically append `-backend` and `-frontend` to this base name to target the correct repositories.

---

### Setting Up GitHub Actions

### Step 1: Set Up GitHub Actions Runner on EC2

1.  On your EC2 instance, create a directory for the runner:
    ```bash
    mkdir actions-runner && cd actions-runner
    ```
2.  Go to your GitHub repository
3.  Click on "Settings" tab
4.  In the left sidebar, click on "Actions" → "Runners"
5.  Click "New self-hosted runner"
6.  Select "Linux" as the operating system
7.  Follow the commands shown on the page to:
    - Download the runner package
    - Extract it
    - Configure and start the runner
8.  Keep the runner service running automatically by configuring it as a systemd service. This ensures the runner starts on boot and restarts if it crashes.
    a. Navigate to the runner directory:
    ```bash
      cd ~/actions-runner
    ```
    b. Install the service:
    ```bash
      sudo ./svc.sh install
    ```
    c. Start the service:
    ```bash
      sudo ./svc.sh start
    ```
    d. Check the status:
    ```bash
      sudo ./svc.sh status
    ```
    If you encounter issues with `needrestart` interfering, you may need to configure it to ignore the runner service by running:
    ```bash
      echo '$nrconf{override_rc}{qr(^actions\.runner\..+\.service$)} = 0;' | sudo tee /etc/needrestart/conf.d/actions_runner_services.conf
    ```

### Step 2: Add GitHub Secrets

In your GitHub repository, add these secrets. These values will be used by the GitHub Actions workflow and injected into your Docker containers at deployment time.

1.  Name: `EC2_HOST`
    Value: Your EC2 instance's public IP address
2.  Name: `EC2_USER`
    Value: `ubuntu`
3.  Name: `AWS_ACCESS_KEY_ID`
    Value: The access key ID you copied from your IAM user.
4.  Name: `AWS_SECRET_ACCESS_KEY`
    Value: The secret access key you copied from your IAM user.
5.  Name: `AWS_REGION`
    Value: The AWS region where your ECR repository is located (e.g., `us-east-1`).
6.  Name: `AWS_ACCOUNT_ID`
    Value: Your 12-digit AWS account ID.
7.  Name: `ECR_REPOSITORY`
    Value: The base name of your ECR repositories (e.g., `yaliaero`).
8.  Name: `JWT_SECRET`
    Value: A secure random string for verification tokens (e.g., generate one at [https://randomkeygen.com/](https://randomkeygen.com/))
9.  Name: `MONGO_URI`
    Value: The full DocumentDB connection string (e.g., mongodb://...)
10. Name: `ADMIN_EMAIL`
    Value: Your admin email that wanted to be admin while creating the ec2 instance
11. Name: `ADMIN_PASSWORD`
    Value: Your admin password that your organization wanted to use
12. Name: `S3_BUCKET`
    Value: The name of your S3 bucket
13. Name: `VERIFICATION_TOKEN_SECRET`
    Value: A secure random string for verification tokens
14. Name: `ASSOCIATE_VERIFICATION_TOKEN_SECRET`
    Value: A secure random string for associate verification tokens
15. Name: `EMAIL_HOST`
    Value: `smtp.gmail.com`
16. Name: `EMAIL_PORT`
    Value: `587`
17. Name: `EMAIL_USER`
    Value: Your email address for sending emails
18. Name: `EMAIL_PASS`
    Value: Your email password or app-specific password
19. Name: `FRONTEND_BASE_URL`
    Value: The public URL of your deployed frontend (e.g., `https://yourdomain.com`)
20. Name: `NODE_ENV`
    Value: `production`

**Note on Email Service Ports**: You do not need to open port 587 (or any other email port) in your EC2 security group's **inbound** rules. Your application makes an **outbound** connection to the email server, and all outbound traffic is allowed by default. This setup is secure and correct.

### Step 3: Understanding the GitHub Actions Workflow

The project includes a GitHub Actions workflow file at `.github/workflows/deploy.yml`. Here's what it does:

```yaml
name: Deploy YaliAero MERN Application to AWS ECR

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and Tag Backend Image
        run: docker build -t ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}_backend:latest ./server

      - name: Build and Tag Frontend Image
        run: docker build -t ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}_frontend:latest ./client-vite

      - name: Push Images to ECR
        run: |
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}_backend:latest
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}_frontend:latest

  deploy:
    needs: build
    runs-on: self-hosted
    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
          
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Checkout Source (for docker-compose.yml)
        uses: actions/checkout@v4

      - name: Add User to Docker Group
        run: sudo usermod -aG docker $USER

      - name: Login to ECR with Docker
        run: |
          aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | sudo docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com

      - name: Install Docker Compose
        run: sudo apt-get update && sudo apt-get install -y docker-compose -o Acquire::ForceIPv4=true

      # inside your deploy job that runs on the self-hosted runner
      - name: Write .env for app
        run: |
          cat > ./docker-compose.env <<'EOF'
          AWS_ACCOUNT_ID=${{ secrets.AWS_ACCOUNT_ID }}
          AWS_REGION=${{ secrets.AWS_REGION }}
          ECR_REPOSITORY=${{ secrets.ECR_REPOSITORY }}
          MONGO_URI=${{ secrets.MONGO_URI }}
          JWT_SECRET=${{ secrets.JWT_SECRET }}
          PORT=${{ secrets.PORT }}
          S3_BUCKET=${{ secrets.S3_BUCKET }}
          VERIFICATION_TOKEN_SECRET=${{ secrets.VERIFICATION_TOKEN_SECRET }}
          ASSOCIATE_VERIFICATION_TOKEN_SECRET=${{ secrets.ASSOCIATE_VERIFICATION_TOKEN_SECRET }}
          EMAIL_HOST=${{ secrets.EMAIL_HOST }}
          EMAIL_PORT=${{ secrets.EMAIL_PORT }}
          EMAIL_USER=${{ secrets.EMAIL_USER }}
          EMAIL_PASS=${{ secrets.EMAIL_PASS }}
          FRONTEND_BASE_URL=${{ secrets.FRONTEND_BASE_URL }}
          NODE_ENV=${{ secrets.NODE_ENV }}
          ADMIN_EMAIL=${{ secrets.ADMIN_EMAIL }}
          ADMIN_PASSWORD=${{ secrets.ADMIN_PASSWORD }}
          AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}
          EOF
          chmod 600 ./docker-compose.env

      - name: Stop and Remove Old Containers
        run: |
          sudo docker-compose --env-file ./docker-compose.env -f docker-compose.yml down --volumes --remove-orphans || true

      - name: Stop and remove the old images from EC2 instance
        run: docker images -f "dangling=true" -q | xargs -r docker rmi -f

      - name: Check Docker Container Status
        run: sudo docker ps -a

      - name: Forcefully free up port 80
        run: sudo fuser -k 80/tcp || true

      # then use this env file with docker-compose
      - name: Start containers
        run: |
          sudo docker-compose --env-file ./docker-compose.env -f docker-compose.yml pull
          sudo docker-compose --env-file ./docker-compose.env -f docker-compose.yml up -d --remove-orphans
```

---

### Deploying Your Application

### Step 1: Set Up Docker Network

On your EC2 instance, create a Docker network for container communication:

```bash
docker network create mern-network
```

### Step 2: Configure GitHub Actions Runner

Make sure your GitHub Actions runner is properly configured and running. You can check its status with:

Start the service:

```bash
   sudo ./svc.sh start
```

### Step 3: Understanding Container Management

The deployment process is now fully automated through GitHub Actions:

1.  **Container Naming**:
    - Backend container: `yaliaero-backend-container`
    - Frontend container: `yaliaero-frontend-container`
2.  **Network Configuration**:
    - All containers use the `mern-network` Docker network
    - Frontend can access backend via `http://backend:5000` (as defined in `docker-compose.yml`)
3.  **Environment Variables**:
    - Backend receives `JWT_SECRET`, `MONGO_URI`, `PORT`, AWS credentials, S3 bucket name, verification token secrets, email service credentials, `FRONTEND_BASE_URL`, and `NODE_ENV` from GitHub Secrets (passed to `docker-compose.yml` and then to the container).
    - Frontend is configured with the correct API URL (`VITE_API_BASE_URL`).
4.  **Port Mapping**:
    - Frontend is accessible on port 80
    - Backend is accessible on port 5000 (within the EC2 instance, not directly exposed to public unless configured otherwise)
5.  **Container Lifecycle**:
    - Old containers are automatically removed by `docker-compose down`
    - New containers are created with fresh images by `docker-compose up -d`
    - Containers run in detached mode

### Step 4: Configure Domain and SSL

1.  **Configure Your Domain**:

    - Go to your domain registrar's website
    - Add an A record pointing to your EC2 public IP
    - Wait for DNS propagation (can take up to 48 hours)

    ### Using AWS Route 53 for Domain Management

    AWS Route 53 is a highly available and scalable Domain Name System (DNS) web service. If your domain is registered with AWS or another registrar, you can use Route 53 to manage its DNS records.

    1.  **Register a New Domain in Route 53 (if you don't have one)**:

        - Navigate to the Route 53 service in the AWS Management Console.
        - In the left sidebar, click on "Registered domains" and then "Register domain".
        - Enter the domain name you want (e.g., [suspicious link removed]) and click "Check".
        - If the domain is available, follow the prompts to complete the registration process. This includes providing your contact information and payment details.

    2.  **Create a Hosted Zone in Route 53**:

        - Navigate to the Route 53 service in the AWS Management Console.
        - Click on "Hosted zones" in the left sidebar.
        - Click "Create hosted zone".
        - Enter your domain name (e.g., `yourdomain.com`).
        - Select "Public hosted zone".
        - Click "Create hosted zone".

    3.  **(optional if you purchase from other domain server)Update Your Domain Registrar's Name Servers**:

        - After creating the hosted zone, Route 53 will provide you with a set of Name Server (NS) records. These typically look like `ns-xxxx.awsdns-xx.org.`, `ns-xxxx.awsdns-xx.net.`, etc.
        - Go to your domain registrar's website (where you purchased your domain name).
        - Find the DNS management or Name Server settings for your domain.
        - Replace the existing Name Servers with the ones provided by Route 53.
        - **Note**: DNS changes can take some time to propagate across the internet (up to 48 hours, though often much faster).

    4.  **Create an 'A' Record in Route 53**:

        - In your Route 53 hosted zone, click "Create record".
        - For "Record name", leave it blank if you are configuring the root domain (e.g., `yourdomain.com`), or enter a subdomain (e.g., `www`).
        - For "Record type", select "A - Routes traffic to an IPv4 address and some AWS resources".
        - For "Value", enter your EC2 instance's Public IPv4 address.
        - For "TTL (Seconds)", you can use the default or set it to a value like 300.
        - Click "Create records".

    By using Route 53, you centralize your domain management within AWS, which can simplify the deployment process, especially when integrating with other AWS services.

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

### Step 5: Verify Deployment

1.  Open a web browser
2.  Go to `https://yourdomain.com` (notice HTTPS)
3.  Verify that:
    - The site loads securely (green padlock in browser)
    - You can register and login
    - The application functions correctly
    - MongoDB connections work (you can create and retrieve data)

---

### Automatic Deployments

Now that you've set up GitHub Actions with a self-hosted runner, any time you push changes to the `main` branch of your repository, GitHub Actions will automatically:

1.  Build and push new Docker images to ECR (build job)
2.  Pull and deploy new containers (deploy job)

To make changes to your application:

1.  Make changes to your code locally
2.  Commit the changes:
    ```bash
    git add .
    git commit -m "Initial commit for YaliAero MERN app"
    ```
3.  Push to GitHub:
    ```bash
    git push origin main
    ```
4.  Wait a few minutes for GitHub Actions to complete both jobs

---

### Troubleshooting

### Issue: GitHub Actions Runner Not Working

1.  Check runner status on EC2:
    ```bash
    sudo ./svc.sh status
    ```
2.  If the runner is not running or needs to be started/restarted:
    ```bash
    sudo ./svc.sh start
    ```
    If you encounter issues with `needrestart` interfering, you may need to configure it to ignore the runner service by running:
    ```bash
    echo '$nrconf{override_rc}{qr(^actions.runner..+.service$)} = 0;' | sudo tee /etc/needrestart/conf.d/actions_runner_services.conf
    ```
3.  Verify runner is online in GitHub:
    - Go to repository Settings → Actions → Runners
    - Check runner status is "Idle" or "Active"

### Issue: Docker Commands Not Working

1.  Make sure Docker is installed and running:
    ```bash
    sudo systemctl status docker
    ```
2.  If not running, start it:
    ```bash
    sudo systemctl start docker
    ```
3.  Make sure your user is in the docker group:
    ```bash
    groups
    ```
    If `docker` is not listed, run:
    ```bash
    sudo usermod -aG docker $USER
    ```
    Then log out and log back in.

### Issue: Application Not Accessible

1.  Check that containers are running:
    ```bash
    docker ps
    ```
2.  Check container logs:
    ```bash
    docker logs yaliaero-frontend-container
    docker logs yaliaero-backend-container
    ```
3.  Verify Docker network exists:
    ```bash
    docker network ls | grep mern-network
    ```
4.  Verify that the security group allows traffic on ports 80 and 443

### Issue: MongoDB Connection Problems

1.  Check MongoDB service status:
    ```bash
    sudo systemctl status mongod
    ```
2.  Verify MongoDB authentication:
    ```bash
    # Try connecting with authentication
    mongosh --username adminUser --password your_secure_password --authenticationDatabase admin
    ```
3.  Check MongoDB logs:
    ```bash
    sudo tail -f /var/log/mongodb/mongod.log
    ```
4.  Verify MongoDB environment variables in backend container:
    ```bash
    docker exec yaliaero-backend-container env | grep MONGO
    ```

### Issue: SSL Certificate Problems

1.  Check certificate status:
    ```bash
    sudo certbot certificates
    ```
2.  Test certificate renewal:
    ```bash
    sudo certbot renew --dry-run
    ```
3.  Verify Nginx SSL configuration:
    ```bash
    sudo nginx -t
    ```
4.  Check SSL certificate paths in container:
    ```bash
    docker exec yaliaero-frontend-container ls -l /etc/letsencrypt/live/
    ```

### Issue: GitHub Actions Build Job Failing

1.  Go to your GitHub repository
2.  Click on "Actions" tab
3.  Click on the failed workflow run
4.  Check the build job logs for:
    - ECR login errors
    - Build errors
    - Push errors
5.  Verify your GitHub secrets:
    - `AWS_ACCESS_KEY_ID`
    - `AWS_SECRET_ACCESS_KEY`
    - `AWS_ACCOUNT_ID`
    - `AWS_REGION`
    - Any other custom environment variables you added

---

### Conclusion

Congratulations\! You've successfully set up the deployment guide for your YaliAero MERN application to AWS EC2 using GitHub Actions and ECR. Your application will be accessible on the internet, and any changes you push to your GitHub repository will be automatically deployed.

### Improving the Robustness of Your Deployment

This guide provides a basic setup for deploying your YaliAero MERN application. To further improve the robustness of your deployment, consider the following:

1.  **Infrastructure as Code (IaC)**: Use tools like Terraform or CloudFormation to automate the creation and management of your AWS infrastructure.
2.  **Monitoring and Logging**: Implement comprehensive monitoring and logging to track the health and performance of your application. Consider using tools like CloudWatch, ELK stack, or Prometheus.
3.  **Scalability**: Design your application to handle increasing traffic and data. Consider using load balancers, auto-scaling groups, and database replication.
4.  **Disaster Recovery**: Implement a plan to recover your application in case of a disaster. Consider using backups, replication, and failover mechanisms.
5.  **Security**: Implement security best practices to protect your application and data. Consider using firewalls, intrusion detection systems, and regular security audits.

### Backend Service Documentation

The backend service is built using Node.js and Express, with Python dependencies managed for specific functionalities. It handles API requests, user authentication, data processing, and interactions with the MongoDB database.

### Key Technologies:

- **Node.js**: Runtime environment for the backend application.
- **Express.js**: Web application framework for Node.js.
- **Python**: Used for specific backend tasks, with dependencies managed via `requirements.txt`.
- **MongoDB**: NoSQL database for storing application data.
- **Docker**: Containerization for packaging and running the backend service.

### Docker Configuration:

- **`server/Dockerfile`**:

  - Uses `node:20-alpine` as the base image.
  - Installs Python, pip, and necessary build tools.
  - Sets up a Python virtual environment (`/opt/venv`).
  - Installs Node.js dependencies (`npm install`).
  - Installs Python dependencies from `requirements.txt` within a virtual environment.
  - Copies application code.
  - Exposes port `5000` for the backend API.
  - Sets `CMD ["npm", "start"]` to run the application.

- **`docker-compose.yml`**:

  - Defines the `backend` service.
  - Builds the image from `./server`.
  - Sets `restart: always` for continuous availability.
  - The backend port `5000` is not directly exposed to the host; it's accessible only within the `mern-network`.
  - Configures environment variables for `NODE_ENV`, `MONGO_URI`, `JWT_SECRET`, AWS credentials, email service, etc., primarily sourced from GitHub Secrets.
  - Depends on the `mongodb` service to ensure the database is available.
  - Connects to the `mern-network`.

### Database Connection:

The backend connects to DocumentDB using the `MONGO_URI` environment variable, which is configured as a GitHub Secret. This ensures secure communication with the database.

### API Access:

The backend API, running on port 5000, is proxied by Nginx. Requests to `/api/` paths are forwarded to the backend container. This setup enhances security by not exposing the backend port directly to the internet.
