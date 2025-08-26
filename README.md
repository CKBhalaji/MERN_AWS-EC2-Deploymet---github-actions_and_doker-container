# Complete Beginner's Guide to Deploying Your YaliAero MERN Application

This guide is designed for absolute beginners who have never used AWS or GitHub Actions before. We'll walk through every single step with detailed instructions to deploy your YaliAero MERN application.

## Table of Contents

1. [Setting Up AWS Account](#setting-up-aws-account)
2. [Creating an EC2 Instance](#creating-an-EC2-instance)
3. [Connecting to Your EC2 Instance](#connecting-to-your-EC2-instance)
4. [Setting Up Your EC2 Instance](#setting-up-your-EC2-instance)
5. [Setting Up GitHub Repository](#setting-up-github-repository)
6. [Setting Up Dockerfiles and Docker Compose](#setting-up-dockerfiles-and-docker-compose)
7. [Setting Up GitHub Actions](#setting-up-github-actions)
8. [Deploying Your Application](#deploying-your-application)
9. [Troubleshooting](#troubleshooting)

## Setting Up AWS Account

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

## Creating an EC2 Instance

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

## Connecting to Your EC2 Instance

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

## Setting Up Your EC2 Instance

Once you're connected to your EC2 instance, run these commands to set up the environment:

```bash
# Update the system
sudo apt update
sudo apt upgrade -y

# Install Docker and Docker Compose
sudo apt install -y docker-ce docker-compose-plugin

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

# Authenticate with Docker Hub (Crucial for private repositories)
docker login

# Install Git
sudo apt install -y git

# Install Certbot for SSL
sudo apt install -y certbot python3-certbot-nginx

```
### (Already Done Explained)why we have to login docker in ec2 instance

After installing Docker on your EC2 instance, and before starting the GitHub Actions runner, you must authenticate with Docker Hub.

Here are the steps you should add:

1.  **SSH into your EC2 instance.**
2.  **Run the `docker login` command.** You will be prompted to enter your Docker Hub username and password.

```bash
docker login
```

3.  **Enter your credentials** when prompted.

After successfully logging in, Docker stores your credentials in a configuration file on the EC2 instance. The GitHub Actions self-hosted runner, running as the `ubuntu` user, will use these stored credentials to authenticate with Docker Hub when it executes the `docker pull` commands in your workflow.

This one-time manual step on your EC2 instance is all you need to do to allow the self-hosted runner to access your private repositories. Your GitHub Actions workflow is already set up to handle the rest of the process, including the build and push, which uses secrets for authentication.

## Nginx Configuration

After setting up your EC2 instance, you need to configure Nginx to serve your application. There are two Nginx configurations used in this project:

1.  **`client-vite/frontend-nginx.conf`**: This configuration is copied into the frontend Docker image. It's used for serving the static frontend files and proxying API requests to the backend container *within the Docker network*.
2.  **`yaliaero.conf`**: This configuration is intended for the Nginx server running directly on the EC2 instance (the host). It acts as a main reverse proxy, handling SSL and routing traffic to the Docker containers.

**For the EC2 instance deployment:**

You will use the `yaliaero.conf` file. Follow these steps to configure Nginx on your EC2 instance:

1.  Connect to your EC2 instance using SSH.
2.  Use a text editor to create a new file in the `/etc/nginx/conf.d/` directory. We recommend using `vim`, which is a powerful text editor commonly found on Linux systems. To use `vim`, follow these steps:

    a.  Open a new file in `vim` with the following command: `sudo vim /etc/nginx/conf.d/yaliaero.conf`
    b.  Press the `i` key to enter insert mode. This will allow you to type text into the file.
    c.  Copy the Nginx configuration from this guide and paste it into the file.
    d.  Press the `Esc` key to exit insert mode.
    e.  Type `:wq` and press `Enter` to save the file and exit `vim`.

    If you prefer to use `nano`, you can use the following command: `sudo nano /etc/nginx/conf.d/yaliaero.conf`. Nano is a simpler text editor that is easier to use for beginners.
3.  Copy the Nginx configuration from this guide and paste it into the file.
4.  Save the file and exit the text editor.
5.  Test the Nginx configuration: `sudo nginx -t`
6.  If the configuration is correct, reload Nginx: `sudo systemctl reload nginx`

### How the Nginx Configuration Works

Nginx acts as a reverse proxy in this setup, handling incoming requests and forwarding them to the appropriate backend servers. Here's a breakdown of how the configuration works:

*   **server blocks**: The `server` blocks define the configuration for different domains or subdomains. In this case, we have two server blocks: one for HTTP (port 80) and one for HTTPS (port 443). The HTTP server block redirects all traffic to HTTPS.
*   **location blocks**: The `location` blocks define how Nginx handles requests to different paths.
    *   The `location /` block handles all requests to the root path and forwards them to the frontend container (`frontend`) on port 80.
    *   The `location /api/` block handles all requests to paths starting with `/api/` and forwards them to the backend container.
*   **proxy_pass directive**: The `proxy_pass http://backend:5000;` directive specifies the address of the backend server to which the request should be forwarded. The line tells Nginx to send the request to the backend container at its internal address on port 5000. This communication happens securely inside the private Docker network and does **not** expose port 5000 to the public internet.
*   **proxy_set_header directives**: The `proxy_set_header` directives pass information about the original request to the backend servers. This information includes the host, the client's IP address, and the protocol used (HTTP or HTTPS).
*   **add_header Cache-Control directive**: The `add_header Cache-Control` directive sets the cache control headers for static assets served by the frontend. This tells the browser how long to cache these assets, which can improve performance.
*   **error_page directive**: The `error_page` directive handles errors. In this case, it specifies that if an error occurs (500, 502, 503, or 504), Nginx should display the `50x.html` file.

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
3.  Update the `server_name` directives in both the HTTP and HTTPS server blocks to use your custom domain name. For example:

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
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    ```
5.  Save the file and exit the text editor.
6.  Test the Nginx configuration: `sudo nginx -t`
7.  If the configuration is correct, reload Nginx: `sudo systemctl reload nginx`

## Setting Up GitHub Repository

### Step 1: Create a GitHub Account (if you don't have one)

1. Go to [https://github.com/](https://github.com/)
2. Click "Sign up"
3. Follow the instructions to create your account

### Step 2: Create a New Repository

1. Click the "+" icon in the top-right corner
2. Select "New repository"
3. Enter a repository name (e.g., "YaliAero_Website")
4. Choose "Public" or "Private"
5. Click "Create repository"

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
git remote add origin https://github.com/your-username/YaliAero_Website.git

# Push to GitHub
git push -u origin main
```

Replace "your-username" with your actual GitHub username.

## Setting Up Dockerfiles and Docker Compose

We will containerize your frontend and backend applications using Docker.  The deployment process utilizes a multi-container architecture managed by `docker-compose.yml`.  The `deploy.yml` file uses `docker-compose up --detach --force-recreate` to ensure that the latest images are always used and old containers are removed.

### Step 1: Create `server/Dockerfile`

This Dockerfile builds the Node.js backend application, including Python dependencies managed within a virtual environment.  The virtual environment ensures that Python dependencies are isolated from the system's Python installation.

Create a file named `Dockerfile` inside the `server/` directory with the following content:

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:20-alpine

# Set the working directory in the container
WORKDIR /app

# Install Python and pip, and build dependencies
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

# Define the command to run the application
# Ensure Python from venv is available in the PATH for any Python scripts called by Node.js
ENV PATH="/opt/venv/bin:$PATH"

# Define the command to run the application
CMD ["npm", "start"]
```

### Step 2: Create `client-vite/Dockerfile`

This Dockerfile builds the React frontend application and serves it using Nginx.

Create a file named `Dockerfile` inside the `client-vite/` directory with the following content:

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:20-alpine as build

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
RUN VITE_API_BASE_URL=/api npm run build
# RUN npm run build

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
npm-debug.log
.env
data/logs.sqlite
data/cache/
.git
.gitignore
README.md
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

### Step 4: Create `docker-compose.yml`

This file defines and runs the multi-container Docker application.  It uses environment variable substitution for sensitive information, which should be provided at runtime from the environment where `docker-compose up` is executed (e.g., from GitHub Actions secrets or a local `.env` file).

Create a file named `docker-compose.yml` in the root of your project directory (`YaliAero_Website/`).

```yaml
version: '3.8'

services:
  backend:
    build: ./server
    container_name: yaliaero-backend-container
    restart: always
    environment:
      NODE_ENV: ${NODE_ENV}
      MONGO_URI: mongodb://adminUser:${MONGO_PASSWORD}@mongodb:27017/yaliaero_db?authSource=admin
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
    depends_on:
      - mongodb
    networks:
      - mern-network

  frontend: 
    build: ./client-vite
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

  mongodb:
    image: mongo:6.0
    container_name: yaliaero-mongodb
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: adminUser
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD} 
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD}
    volumes:
      - mongodb_data:/data/db
    networks:
      - mern-network

volumes:
  mongodb_data:

networks:
  mern-network:
    driver: bridge
```

Data Persistence for MongoDB with Docker Volumes:

When running MongoDB as a Docker container using `docker-compose.yml`, its data is stored in a Docker volume.  In this setup, a named volume called `mongodb_data` is defined.  This ensures data persistence even if the container is restarted or recreated.

**Data Persistence for MongoDB with Docker Volumes**:

When you run MongoDB as a Docker container using `docker-compose.yml`, its data is stored in a Docker volume. In this setup, we've defined a named volume called `mongodb_data`.

*   **`volumes: - mongodb_data:/data/db`**: This line in the `mongodb` service definition tells Docker to mount the `mongodb_data` volume to the `/data/db` directory inside the MongoDB container. This `/data/db` directory is where MongoDB stores its data files.
*   **`volumes: mongodb_data:`**: This top-level `volumes` definition declares `mongodb_data` as a named volume. Docker manages this volume, and its data persists on the host machine (your EC2 instance) even if the `yaliaero-mongodb` container is stopped, removed, or recreated.

This means your MongoDB data will **not** be erased when you restart or update your Docker containers. It will remain safely stored in the `mongodb_data` volume on your EC2 instance.

## Setting Up GitHub Actions

### Step 1: Set Up GitHub Actions Runner on EC2

1. On your EC2 instance, create a directory for the runner:
```bash
mkdir actions-runner && cd actions-runner
```

2. Go to your GitHub repository
3. Click on "Settings" tab
4. In the left sidebar, click on "Actions" → "Runners"
5. Click "New self-hosted runner"
6. Select "Linux" as the operating system
7. Follow the commands shown on the page to:
   - Download the runner package
   - Extract it
   - Configure and start the runner

8. Keep the runner service running automatically by configuring it as a systemd service. This ensures the runner starts on boot and restarts if it crashes.

   a.  Navigate to the runner directory:
      ```bash
      cd ~/actions-runner
      ```
   b.  Install the service:
      ```bash
      sudo ./svc.sh install
      ```
   c.  Start the service:
      ```bash
      sudo ./svc.sh start
      ```
   d.  Check the status:
      ```bash
      sudo ./svc.sh status
      ```
      If you encounter issues with `needrestart` interfering, you may need to configure it to ignore the runner service by running:
      ```bash
      echo '$nrconf{override_rc}{qr(^actions\.runner\..+\.service$)} = 0;' | sudo tee /etc/needrestart/conf.d/actions_runner_services.conf
      ```

### Step 2: Set Up Docker Hub

1. Go to [https://hub.docker.com/](https://hub.docker.com/)

**Note**: Instead of using a self-hosted runner, you can also use a GitHub-hosted runner with an SSH action for deployment. This approach requires less manual setup on the EC2 instance.

2. Click "Sign Up" if you don't have an account
3. Create two repositories:
   - Click "Create Repository"
   - Name the first one `yaliaero-backend`
   - Set visibility to "Public"
   - Click "Create"
   - Repeat for `yaliaero-frontend`

### Step 3: Add GitHub Secrets

In your GitHub repository, add these secrets. These values will be used by the GitHub Actions workflow and injected into your Docker containers at deployment time.

1. Name: `EC2_HOST`
   Value: Your EC2 instance's public IP address

2. Name: `EC2_USER`
   Value: `ubuntu`

3. Name: `JWT_SECRET`
   Value: A secure random string for verification tokens (e.g., generate one at [https://randomkeygen.com/](https://randomkeygen.com/))

4. Name: `DOCKER_USERNAME`
   Value: Your Docker Hub username

5. Name: `DOCKER_PASSWORD`
   Value: Your Docker Hub password or access token (recommended)

6. Name: `MONGO_USER`
   Value: Your MongoDB username (e.g., `adminUser`)

7. Name: `MONGO_PASSWORD`
   Value: Your MongoDB password (the one you set during MongoDB setup)

8. Name: `AWS_ACCESS_KEY_ID`
   Value: Your AWS Access Key ID

9. Name: `AWS_SECRET_ACCESS_KEY`
   Value: Your AWS Secret Access Key

10. Name: `AWS_REGION`
    Value: `ap-south-1`

11. Name: `S3_BUCKET`
    Value: The name of your S3 bucket

12. Name: `VERIFICATION_TOKEN_SECRET`
    Value: A secure random string for verification tokens

13. Name: `ASSOCIATE_VERIFICATION_TOKEN_SECRET`
    Value: A secure random string for associate verification tokens

14. Name: `EMAIL_HOST`
    Value: `smtp.gmail.com`

15. Name: `EMAIL_PORT`
    Value: `587`

16. Name: `EMAIL_USER`
    Value: Your email address for sending emails

17. Name: `EMAIL_PASS`
    Value: Your email password or app-specific password

18. Name: `FRONTEND_BASE_URL`
    Value: The public URL of your deployed frontend (e.g., `https://yourdomain.com`)

19. Name: `NODE_ENV`
    Value: `production`

**Note on Email Service Ports**: You do not need to open port 587 (or any other email port) in your EC2 security group's **inbound** rules. Your application makes an **outbound** connection to the email server, and all outbound traffic is allowed by default. This setup is secure and correct.

### Step 4: Understanding the GitHub Actions Workflow

The project includes a GitHub Actions workflow file at `.github/workflows/deploy.yml`. Here's what it does:

**Note**: If you choose to use a GitHub-hosted runner with an SSH action for deployment, you will need to modify this workflow file to include the necessary SSH configuration. This may involve changing the `runs-on` value for the `deploy` job, adding the SSH action, and updating the steps to deploy the application to the EC2 instance. You may also need to add additional secrets, such as the SSH private key.

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
    runs-on: self-hosted
    steps:
      - name: Checkout Source (for docker-compose.yml)
        uses: actions/checkout@v4

      - name: Pull Images from Docker Hub
        run: |
          docker pull ${{ secrets.DOCKER_USERNAME }}/yaliaero-backend:latest
          docker pull ${{ secrets.DOCKER_USERNAME }}/yaliaero-frontend:latest

      - name: Stop and Remove Old Containers
        run: |
          docker-compose -f docker-compose.yml down || true

      - name: Run Docker Compose
        run: |
          docker-compose -f docker-compose.yml up -d
        env:
          MONGO_URI: mongodb://adminUser:${{ secrets.MONGO_PASSWORD }}@mongodb:27017/yaliaero_db?authSource=admin
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          PORT: 5000
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
          S3_BUCKET: ${{ secrets.S3_BUCKET }}
          VERIFICATION_TOKEN_SECRET: ${{ secrets.VERIFICATION_TOKEN_SECRET }}
          ASSOCIATE_VERIFICATION_TOKEN_SECRET: ${{ secrets.ASSOCIATE_VERIFICATION_TOKEN_SECRET }}
          EMAIL_HOST: ${{ secrets.EMAIL_HOST }}
          EMAIL_PORT: ${{ secrets.EMAIL_PORT }}
          EMAIL_USER: ${{ secrets.EMAIL_USER }}
          EMAIL_PASS: ${{ secrets.EMAIL_PASS }}
          FRONTEND_BASE_URL: ${{ secrets.FRONTEND_BASE_URL }}
          NODE_ENV: production
```

## Deploying Your Application

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

1. **Container Naming**:
   - Backend container: `yaliaero-backend-container`
   - Frontend container: `yaliaero-frontend-container`

2. **Network Configuration**:
   - All containers use the `mern-network` Docker network
   - Frontend can access backend via `http://backend:5000` (as defined in `docker-compose.yml`)

3. **Environment Variables**:
   - Backend receives `JWT_SECRET`, `MONGO_URI`, `PORT`, AWS credentials, S3 bucket name, verification token secrets, email service credentials, `FRONTEND_BASE_URL`, and `NODE_ENV` from GitHub Secrets (passed to `docker-compose.yml` and then to the container).
   - Frontend is configured with the correct API URL (`VITE_API_BASE_URL`).

4. **Port Mapping**:
   - Frontend is accessible on port 80
   - Backend is accessible on port 5000 (within the EC2 instance, not directly exposed to public unless configured otherwise)

5. **Container Lifecycle**:
   - Old containers are automatically removed by `docker-compose down`
   - New containers are created with fresh images by `docker-compose up -d`
   - Containers run in detached mode

### Step 4: Configure Domain and SSL

1. **Configure Your Domain**:
   - Go to your domain registrar's website
   - Add an A record pointing to your EC2 public IP
   - Wait for DNS propagation (can take up to 48 hours)

   ### Using AWS Route 53 for Domain Management

   AWS Route 53 is a highly available and scalable Domain Name System (DNS) web service. If your domain is registered with AWS or another registrar, you can use Route 53 to manage its DNS records.

   1.  **Register a New Domain in Route 53 (if you don't have one)**:
      *   Navigate to the Route 53 service in the AWS Management Console.
      *   In the left sidebar, click on "Registered domains" and then "Register domain".
      *   Enter the domain name you want (e.g., yourdomain.com) and click "Check".
      *   If the domain is available, follow the prompts to complete the registration process. This includes providing your contact information and payment details.

   2.  **Create a Hosted Zone in Route 53**:
      *   Navigate to the Route 53 service in the AWS Management Console.
      *   Click on "Hosted zones" in the left sidebar.
      *   Click "Create hosted zone".
      *   Enter your domain name (e.g., `yourdomain.com`).
      *   Select "Public hosted zone".
      *   Click "Create hosted zone".

   3.  **(optional if you purchase from other domain server)Update Your Domain Registrar's Name Servers**:
      *   After creating the hosted zone, Route 53 will provide you with a set of Name Server (NS) records. These typically look like `ns-xxxx.awsdns-xx.org.`, `ns-xxxx.awsdns-xx.net.`, etc.
      *   Go to your domain registrar's website (where you purchased your domain name).
      *   Find the DNS management or Name Server settings for your domain.
      *   Replace the existing Name Servers with the ones provided by Route 53.
      *   **Note**: DNS changes can take some time to propagate across the internet (up to 48 hours, though often much faster).

   4.  **Create an 'A' Record in Route 53**:
      *   In your Route 53 hosted zone, click "Create record".
      *   For "Record name", leave it blank if you are configuring the root domain (e.g., `yourdomain.com`), or enter a subdomain (e.g., `www`).
      *   For "Record type", select "A - Routes traffic to an IPv4 address and some AWS resources".
      *   For "Value", enter your EC2 instance's Public IPv4 address.
      *   For "TTL (Seconds)", you can use the default or set it to a value like 300.
      *   Click "Create records".

   By using Route 53, you centralize your domain management within AWS, which can simplify the deployment process, especially when integrating with other AWS services.

2. **Set Up SSL Certificate**:
   ```bash
      # Get SSL certificate for your domain
      sudo certbot --nginx -d yourdomain.com
      # Follow the prompts to configure HTTPS
   ````

3. **Verify SSL Configuration**:
   ```bash
      # Test SSL renewal
      sudo certbot renew --dry-run
   ```

### Step 5: Verify Deployment

1. Open a web browser
2. Go to `https://yourdomain.com` (notice HTTPS)
3. Verify that:
   - The site loads securely (green padlock in browser)
   - You can register and login
   - The application functions correctly
   - MongoDB connections work (you can create and retrieve data)

## Automatic Deployments

Now that you've set up GitHub Actions with a self-hosted runner, any time you push changes to the `main` branch of your repository, GitHub Actions will automatically:

1. Build and push new Docker images (build job)
2. Pull and deploy new containers (deploy job)

To make changes to your application:

1. Make changes to your code locally
2. Commit the changes:
   ```bash
      git add .
      git commit -m "Initial commit for YaliAero MERN app"
   ```
3. Push to GitHub:
   ```bash
   git push origin main
   ```
4. Wait a few minutes for GitHub Actions to complete both jobs

## Troubleshooting


### Issue: GitHub Actions Runner Not Working

1. Check runner status on EC2:
   ```bash
      sudo ./svc.sh status
   ````

2. If the runner is not running or needs to be started/restarted:
   ```bash
      sudo ./svc.sh start
   ```
   If you encounter issues with `needrestart` interfering, you may need to configure it to ignore the runner service by running:
   ```bash
      echo '$nrconf{override_rc}{qr(^actions.runner..+.service$)} = 0;' | sudo tee /etc/needrestart/conf.d/actions_runner_services.conf
   ```

3. Verify runner is online in GitHub:
   - Go to repository Settings → Actions → Runners
   - Check runner status is "Idle" or "Active"

### Issue: Docker Commands Not Working

1. Make sure Docker is installed and running:
   ```bash
   sudo systemctl status docker
   ```
2. If not running, start it:
   ```bash
   sudo systemctl start docker
   ```
3. Make sure your user is in the docker group:
   ```bash
   groups
   ```
   If `docker` is not listed, run:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Then log out and log back in.

### Issue: Application Not Accessible

1. Check that containers are running:
   ```bash
   docker ps
   ```
2. Check container logs:
   ```bash
   docker logs yaliaero-frontend-container
   docker logs yaliaero-backend-container
   ```
3. Verify Docker network exists:
   ```bash
   docker network ls | grep mern-network
   ```
4. Verify that the security group allows traffic on ports 80 and 443

### Issue: MongoDB Connection Problems

1. Check MongoDB service status:
   ```bash
   sudo systemctl status mongod
   ```
2. Verify MongoDB authentication:
   ```bash
   # Try connecting with authentication
   mongosh --username adminUser --password your_secure_password --authenticationDatabase admin
   ```
3. Check MongoDB logs:
   ```bash
   sudo tail -f /var/log/mongodb/mongod.log
   ```
4. Verify MongoDB environment variables in backend container:
   ```bash
   docker exec yaliaero-backend-container env | grep MONGO
   ```

### Issue: SSL Certificate Problems

1. Check certificate status:
   ```bash
   sudo certbot certificates
   ```
2. Test certificate renewal:
   ```bash
   sudo certbot renew --dry-run
   ```
3. Verify Nginx SSL configuration:
   ```bash
   sudo nginx -t
   ```
4. Check SSL certificate paths in container:
   ```bash
   docker exec yaliaero-frontend-container ls -l /etc/letsencrypt/live/
   ```

### Issue: GitHub Actions Build Job Failing

1. Go to your GitHub repository
2. Click on "Actions" tab
3. Click on the failed workflow run
4. Check the build job logs for:
   - Docker Hub login errors
   - Build errors
   - Push errors
5. Verify your GitHub secrets:
   - `DOCKER_USERNAME`
   - `DOCKER_PASSWORD`
   - `JWT_SECRET`
   - `MONGO_PASSWORD`
   - Any other custom environment variables you added

## Conclusion

Congratulations! You've successfully set up the deployment guide for your YaliAero MERN application to AWS EC2 using GitHub Actions. Your application will be accessible on the internet, and any changes you push to your GitHub repository will be automatically deployed.

## Improving the Robustness of Your Deployment

This guide provides a basic setup for deploying your YaliAero MERN application. To further improve the robustness of your deployment, consider the following:

1.  **Infrastructure as Code (IaC)**: Use tools like Terraform or CloudFormation to automate the creation and management of your AWS infrastructure.
2.  **Monitoring and Logging**: Implement comprehensive monitoring and logging to track the health and performance of your application. Consider using tools like CloudWatch, ELK stack, or Prometheus.
3.  **Scalability**: Design your application to handle increasing traffic and data. Consider using load balancers, auto-scaling groups, and database replication.
4.  **Disaster Recovery**: Implement a plan to recover your application in case of a disaster. Consider using backups, replication, and failover mechanisms.
5.  **Security**: Implement security best practices to protect your application and data. Consider using firewalls, intrusion detection systems, and regular security audits.

## Backend Service Documentation

The backend service is built using Node.js and Express, with Python dependencies managed for specific functionalities. It handles API requests, user authentication, data processing, and interactions with the MongoDB database.

### Key Technologies:
*   **Node.js**: Runtime environment for the backend application.
*   **Express.js**: Web application framework for Node.js.
*   **Python**: Used for specific backend tasks, with dependencies managed via `requirements.txt`.
*   **MongoDB**: NoSQL database for storing application data.
*   **Docker**: Containerization for packaging and running the backend service.

### Docker Configuration:

*   **`server/Dockerfile`**:
    *   Uses `node:20-alpine` as the base image.
    *   Installs Python, pip, and necessary build tools.
    *   Sets up a Python virtual environment (`/opt/venv`).
    *   Installs Node.js dependencies (`npm install`).
*   Installs Python dependencies from `requirements.txt` within a virtual environment.
    *   Copies application code.
    *   Exposes port `5000` for the backend API.
    *   Sets `CMD ["npm", "start"]` to run the application.

*   **`docker-compose.yml`**:
    *   Defines the `backend` service.
    *   Builds the image from `./server`.
    *   Sets `restart: always` for continuous availability.
    *   The backend port `5000` is not directly exposed to the host; it's accessible only within the `mern-network`.
    *   Configures environment variables for `NODE_ENV`, `MONGO_URI`, `JWT_SECRET`, AWS credentials, email service, etc., primarily sourced from GitHub Secrets.
    *   Depends on the `mongodb` service to ensure the database is available.
    *   Connects to the `mern-network`.

### Database Connection:

The backend connects to MongoDB using the `MONGO_URI` environment variable, which is configured in `docker-compose.yml` to point to the `mongodb` service within the Docker network:
`mongodb://adminUser:${MONGO_PASSWORD}@mongodb:27017/yaliaero_db?authSource=admin`
This ensures secure and internal communication with the database.

### API Access:

The backend API, running on port 5000, is proxied by Nginx. Requests to `/api/` paths are forwarded to the backend container. This setup enhances security by not exposing the backend port directly to the internet.
