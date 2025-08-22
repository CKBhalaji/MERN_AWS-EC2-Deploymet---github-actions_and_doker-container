# Complete Beginner's Guide to Deploying Your YaliAero MERN Application

This guide is designed for absolute beginners who have never used AWS or GitHub Actions before. We'll walk through every single step with detailed instructions to deploy your YaliAero MERN application.

## Table of Contents

1. [Setting Up AWS Account](#setting-up-aws-account)
2. [Creating an EC2 Instance](#creating-an-ec2-instance)
3. [Connecting to Your EC2 Instance](#connecting-to-your-ec2-instance)
4. [Setting Up Your EC2 Instance](#setting-up-your-ec2-instance)
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
   - SSH: Port 22, Source: Anywhere (0.0.0.0/0)
   - HTTP: Port 80, Source: Anywhere (0.0.0.0/0)
   - HTTPS: Port 443, Source: Anywhere (0.0.0.0/0)
   - Custom TCP: Port 5000, Source: Anywhere (0.0.0.0/0) - *Note: While this opens port 5000 at the network level, your backend's CORS configuration will restrict application-level access to your frontend's URL.*
   - Custom TCP: Port 27017, Source: Anywhere (0.0.0.0/0)

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

# Install Docker
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce
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

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify Docker Compose is working
docker-compose --version

# Install Git
sudo apt install -y git

# Install MongoDB
# Note: If you are using Docker Compose to run MongoDB as a container (as described in "Setting Up Dockerfiles and Docker Compose"), you can skip these manual MongoDB installation steps on the EC2 instance.
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt update
sudo apt install -y mongodb-org

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod

# Configure MongoDB Security
mongosh --eval '
  use admin;
  db.createUser({
    user: "adminUser",
    pwd: "your_secure_password",
    roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
  });
'

# Enable authentication in MongoDB config
sudo nano /etc/mongod.conf
# Add these lines:
# security:
#   authorization: enabled

# Restart MongoDB
sudo systemctl restart mongod

# Install Certbot for SSL
sudo apt install -y certbot python3-certbot-nginx
```

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

We will containerize your frontend and backend applications using Docker.

### Step 1: Create `server/Dockerfile`

This Dockerfile will build your Node.js backend application, including Python dependencies. Create a file named `Dockerfile` inside the `server/` directory with the following content:

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:20-alpine

# Set the working directory in the container
WORKDIR /app

# Install Python and pip
RUN apk add --no-cache python3 py3-pip

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install Node.js dependencies
RUN npm install

# Copy Python requirements and install them
COPY requirements.txt ./
RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 5000

# Define the command to run the application
# Assuming the Node.js server is the primary entry point, and Python scripts are called from Node.js
CMD ["npm", "start"]
```

### Step 2: Create `client-vite/Dockerfile`

This Dockerfile will build your React frontend application and serve it using Nginx. Create a file named `Dockerfile` inside the `client-vite/` directory with the following content:

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
RUN npm run build

# Use Nginx to serve the static files
FROM nginx:alpine

# Copy the built React app to Nginx's public directory
COPY --from=build /app/dist /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Step 3: Create `.dockerignore` files

To optimize your Docker builds and prevent unnecessary files from being copied into your Docker images, create `.dockerignore` files in both your `server/` and `client-vite/` directories.

**`server/.dockerignore`**:
Create a file named `.dockerignore` inside the `server/` directory with the following content:
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
Create a file named `.dockerignore` inside the `client-vite/` directory with the following content:
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

This file will define and run your multi-container Docker application. Create a file named `docker-compose.yml` in the root of your project (`YaliAero_Website/`) with the following content.

**Important**: This `docker-compose.yml` uses environment variable substitution (e.g., `${MONGO_PASSWORD}`). This means that the actual values for these variables will be provided at runtime from the environment where `docker-compose up` is executed (e.g., from GitHub Actions secrets or a local `.env` file). **Do not hardcode sensitive information directly into this file.**

```yaml
version: '3.8'

services:
  backend:
    build: ./server
    container_name: yaliaero-backend-container
    restart: always
    ports:
      - "5000:5000"
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
      VITE_API_BASE_URL: http://backend:5000 # This points to the backend service within the Docker network
    depends_on:
      - backend
    networks:
      - mern-network

  mongodb:
    image: mongo:6.0
    container_name: yaliaero-mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: adminUser
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD} # Use environment variable for MongoDB root password
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

8. Keep the runner service running (you can use `screen` or `tmux` to keep it running after disconnecting):
```bash
# Install screen if not already installed
sudo apt install screen

# Create a new screen session
screen -S github-runner

# Start the runner
./run.sh

# Detach from screen (press Ctrl+A, then D)
```

### Step 2: Set Up Docker Hub

1. Go to [https://hub.docker.com/](https://hub.docker.com/)
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
   Value: A secure random string for JWT authentication (e.g., generate one at [https://randomkeygen.com/](https://randomkeygen.com/))

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

### Step 4: Understanding the GitHub Actions Workflow

The project includes a GitHub Actions workflow file at `.github/workflows/deploy.yml`. Here's what it does:

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

```bash
# If using screen, reattach to the runner session
screen -r github-runner

# Check if the runner is connected and idle
# Press Ctrl+A, then D to detach from screen
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

2. **Set Up SSL Certificate**:
   ```bash
   # Get SSL certificate for your domain
   sudo certbot --nginx -d yourdomain.com
   # Follow the prompts to configure HTTPS
   ```

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
   git commit -m "Your commit message"
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
   # Reattach to screen session
   screen -r github-runner
   ```
2. If runner is not running, start it:
   ```bash
   cd ~/actions-runner
   ./run.sh
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

If you need further assistance, please refer to the AWS documentation or GitHub Actions documentation, or reach out to the community for help.

</final_file_content>

IMPORTANT: For any future changes to this file, use the final_file_content shown above as your reference. This content reflects the current state of the file, including any auto-formatting (e.g., if you used single quotes but the formatter converted them to double quotes). Always base your SEARCH/REPLACE operations on this final version to ensure accuracy.

<environment_details>
# VSCode Visible Files
deployment_readme.md

# VSCode Open Tabs
docker-compose.yml
deployment_readme.md

# Current Time
8/22/2025, 3:07:50 PM (Asia/Calcutta, UTC+5.5:00)

# Context Window Usage
222,973 / 1,048.576K tokens used (21%)

# Current Mode
ACT MODE
</environment_details>
