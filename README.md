
# Travel Memory Application Deployment with AWS Services

## Overview

This guide explains how to deploy the **Travel Memory** MERN-stack application using AWS services. The deployment covers:

- Hosting the application on an EC2 instance  
- Connecting with MongoDB Compass  
- Deploying both Frontend and Backend on EC2  
- Creating multiple instances using AWS Launch Templates  
- Using Nginx to serve the app on port 80  
- Attaching an AWS Load Balancer  
- Connecting a domain using Cloudflare  

---

## Architecture Flow

```
User → Cloudflare Domain → AWS Load Balancer → EC2 Instances → Nginx → Node.js App → MongoDB
```

---

## Setting Up the Application on AWS EC2

### Step 1: Launch EC2 Instance for Backend

- Launch an Ubuntu EC2 instance.
- Connect via SSH.

---

### Step 2: Install Required Packages (Backend Setup)

```bash
sudo apt update -y
sudo apt install nodejs -y
sudo apt install npm -y

nodejs -v
npm -v
```

---

### Step 3: Deploy Backend Code

```bash
sudo bash
cd ~
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
```

---

### Step 4: Create `.env` File

```bash
nano .env
```

Add the following:

```env
PORT=3001
MONGO_URI='ENTER_YOUR_MONGODB_CONNECTION_STRING'
```

Save and exit.

---

### Step 5: Install Dependencies

```bash
npm install
```

---

### Step 6: Start Backend Server

```bash
node index.js
```

Your backend should now be running on:

```
http://<EC2_PUBLIC_IP>:3001

```
<img width="748" height="209" alt="Screenshot 2026-02-01 at 9 33 54 PM" src="https://github.com/user-attachments/assets/ad2f1765-e10b-4e9d-ac42-757fee41a0e8" />


# Connecting the Application to MongoDB Atlas

This guide explains how to set up MongoDB Atlas and connect it to your application and MongoDB Compass.

---

## Steps to Set Up MongoDB on Atlas

### 1. Log in to MongoDB Atlas
- Visit: https://cloud.mongodb.com/
- Log in to your MongoDB Atlas account.

---

### 2. Create Organization and Project
- From the dashboard, create:
  - An **Organization**
  - A **Project** inside that organization

---

### 3. Create a Cluster
- Click **Create Cluster**.
- Name the cluster: `herocluster1`
- Select the **M0 Free Plan**.
- Click **Create Deployment** to deploy the cluster.

---

### 4. Create a Database User
- Go to **Database Access**.
- Click **Add New Database User**.
- Set a **username** and **password**.
- Click **Create DB User**.

---

### 5. Configure Network Access
- Go to **Network Access** from the left panel.
- Click **Add IP Address**.
- Enter:
  ```
  0.0.0.0/0
  ```
  to allow access from all IP addresses.
- Confirm the changes.

---

### 6. Connect MongoDB to Compass
- Go to the **Database** section.
- Click **Connect**.
- Select **Compass** as the connection method.
- Choose **I already have MongoDB Compass** if it's installed.

---

### 7. Copy the Connection String
- Copy the connection string provided by MongoDB Atlas.

Example format:
```
mongodb+srv://<username>:<password>@cluster0.mongodb.net/?retryWrites=true&w=majority
```

---

### 8. Connect via MongoDB Compass
- Open **MongoDB Compass**.
- Paste the connection string into the **New Connection** field.
- Click **Connect**.

---

## You're Connected!
Your MongoDB Atlas database is now successfully connected and ready to use with your application and Compass.

---

### Security Tip
For production environments, avoid using `0.0.0.0/0`. Instead, whitelist only trusted IP addresses.

---
## Setting Up the Application on AWS EC2

### Step 1: Launch EC2 Instance for Frontend

- Launch an Ubuntu EC2 instance.
- Connect via SSH.

---

### Step 2: Install Required Packages (Frontend Setup)

```bash
sudo apt update -y
sudo apt install nodejs -y
sudo apt install npm -y

nodejs -v
npm -v
```

---

### Step 3: Deploy Frontend Code

```bash
sudo bash
cd ~
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/frontend
```

---

### Step 4: Create `.env` File

```bash
nano .env
```

Add the following:

```env
REACT_APP_BACKEND_URL=http://<EC2_PUBLIC_IP>:3001
```

Save and exit.

---

### Step 5: Install Dependencies

```bash
npm install
```

---

### Step 6: Start Frontend Server

```bash
npm start
```

Your frontend should now be running on:

```
http://<EC2_PUBLIC_IP>:3000
```
<img width="857" height="347" alt="Screenshot 2026-02-01 at 9 33 48 PM" src="https://github.com/user-attachments/assets/196037ba-6861-4e07-afac-b774803df6f8" />

<img width="607" height="370" alt="Screenshot 2026-02-01 at 9 36 55 PM" src="https://github.com/user-attachments/assets/2cdad023-6c3e-4d8c-8f53-51e8bc6867bf" />

---

# Creating a Reverse Proxy Using Nginx (Frontend)

This section explains how to configure Nginx as a reverse proxy for the **Travel Memory frontend** running on port `3000`.

---

## Step 1: Install and Set Up Nginx on EC2

```bash
sudo bash
cd ~
apt update -y
apt install nginx -y
```

---

## Step 2: Edit Nginx Configuration

Open the default configuration file:

```bash
nano /etc/nginx/sites-available/default
```

Remove the existing code and paste the following:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://<EC2_PUBLIC_IP>:3000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Save and exit.

---

## Step 3: Restart and Test Nginx

```bash
nginx -t
systemctl reload nginx
```

---

## Step 4: Test in Browser

Open your browser and navigate to:

```text
http://<EC2_PUBLIC_IP>
```
<img width="1512" height="836" alt="Screenshot 2026-02-06 at 11 09 45 PM" src="https://github.com/user-attachments/assets/893dc0c9-9e0a-4031-9945-3b3bb02775ac" />


If your frontend loads successfully, 🎉 your reverse proxy is working!

---

# Creating a Reverse Proxy Using Nginx (Backend)

This section explains how to configure Nginx as a reverse proxy for the **Travel Memory backend** running on port `3001`.

---

## Step 1: Install and Set Up Nginx on EC2

```bash
sudo bash
cd ~
apt update -y
apt install nginx -y
```

---

## Step 2: Edit Nginx Configuration

Open the default configuration file:

```bash
nano /etc/nginx/sites-available/default
```

Remove the existing code and paste the following:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://<EC2_PUBLIC_IP>:3001;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Save and exit.

---

## Step 3: Restart and Test Nginx

```bash
nginx -t
systemctl reload nginx
```

---

## Step 4: Test Backend in Browser or Postman

Open your browser and navigate to:

```text
http://<EC2_PUBLIC_IP>/trip
```
<img width="1505" height="834" alt="Screenshot 2026-02-06 at 11 14 30 PM" src="https://github.com/user-attachments/assets/ff902d35-919f-4713-a4aa-1cb0f1d14000" />

If you receive a valid response, 🎉 your backend reverse proxy is working!

---

# Running Frontend & Backend with Custom Domains

This section shows how to run:

- **Frontend** on: `https://learningtech.store`
- **Backend API** on: `https://api.learningtech.store`

## Step 1: Point Domains to Your EC2 IP

In your domain DNS provider:

| Type | Name | Value (EC2 Public IP) |
|------|------|-----------------------|
| A    | @    | <EC2_PUBLIC_IP>       |
| A    | www  | <EC2_PUBLIC_IP>       |
| A    | api  | <EC2_PUBLIC_IP>       |

Wait a few minutes for DNS to propagate.

---
<img width="1134" height="390" alt="Screenshot 2026-02-07 at 7 23 43 PM" src="https://github.com/user-attachments/assets/ae2dffee-9d3f-4887-969e-1949e8d52912" />


## Step 2: Create Separate Nginx Server Blocks

Create a new config file:

```bash
nano /etc/nginx/sites-available/default
```

Paste the following configuration:

```nginx
# Frontend - learningtech.store
server {
    listen 80;
    server_name learningtech.store www.learningtech.store;

    location / {
        proxy_pass http://<EC2_PUBLIC_IP>:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Backend - api.learningtech.store
server {
    listen 80;
    server_name api.learningtech.store;

    location / {
        proxy_pass http://<EC2_PUBLIC_IP>:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Step 3: Enable the Configuration for Frontend & Backend

```bash
nano /etc/nginx/sites-available/default

nginx -t
systemctl reload nginx
```

---

## Step 4: Test in Browser

- Frontend:  
  ```text
  http://learningtech.store
  ```
  <img width="608" height="525" alt="Screenshot 2026-02-07 at 11 28 59 PM" src="https://github.com/user-attachments/assets/1fb46f96-97ed-4304-bd1b-135f131bdc87" />

- Backend API:  
  ```text
  http://api.learningtech.store
  ```
  <img width="577" height="505" alt="Screenshot 2026-02-07 at 11 29 08 PM" src="https://github.com/user-attachments/assets/86f5e6ed-9242-4b03-b37f-107ee084518b" />

If both load correctly, 🎉 your domains are now routing properly!

---

# Install Certbot and Enable SSL for Frontend

This section explains how to secure your frontend domain using HTTPS with Certbot.

---

## Step 1: Install Certbot

```bash
sudo bash
cd ~
apt update
apt install certbot python3-certbot-nginx -y
```

---

## Step 2: Generate and Configure SSL Certificates

```bash
sudo certbot --nginx -d learningtech.store -d www.learningtech.store
```

Certbot will:
- Verify domain ownership
- Generate SSL certificates
- Automatically update your Nginx configuration to use HTTPS

---

## Final Nginx Configuration After SSL

```nginx
server {
    server_name learningtech.store www.learningtech.store;

    location / {
        proxy_pass http://<EC2_PUBLIC_IP>:3000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/learningtech.store/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/learningtech.store/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = www.learningtech.store) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = learningtech.store) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name learningtech.store www.learningtech.store;
    return 404; # managed by Certbot
}
```
---

## Step 3: Verify DNS Records

```bash
dig +short learningtech.store
dig +short www.learningtech.store
```

Ensure both point to your EC2 public IP.

---

## Step 4: Reload and Test Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 5: Access Your Website Securely

🌐 https://learningtech.store  
🌐 https://www.learningtech.store

<img width="1512" height="580" alt="Screenshot 2026-02-01 at 11 40 38 PM" src="https://github.com/user-attachments/assets/a67a1487-0c65-4c9c-9fca-d9cf3930cf54" />
<img width="1512" height="770" alt="Screenshot 2026-02-01 at 11 40 48 PM" src="https://github.com/user-attachments/assets/17e59bd8-c54b-4e3c-9da0-58a92f121a80" />


If your site loads with a 🔒 lock icon, your SSL setup is successful! 🎉

---

# Install Certbot and Enable SSL for Backend API

This section explains how to secure your Backend subdomain using HTTPS with Certbot.

---

## Step 1: Install Certbot

```bash
sudo bash
cd ~
apt update
apt install certbot python3-certbot-nginx -y
```

---

## Step 2: Generate and Configure SSL Certificates

```bash
sudo certbot --nginx -d api.learningtech.store
```

Certbot will:
- Verify domain ownership
- Generate SSL certificates
- Automatically update your Nginx configuration to use HTTPS

---

## Final Nginx Configuration After SSL

```nginx
server {
    server_name api.learningtech.store;

    location / {
        proxy_pass http://<EC2_PUBLIC_IP>:3001;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.learningtech.store/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.learningtech.store/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = api.learningtech.store) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name api.learningtech.store;
    return 404; # managed by Certbot

}
```

---

## Step 3: Verify DNS Records

```bash
dig +short api.learningtech.store
```

Ensure both point to your EC2 public IP.

---

## Step 4: Reload and Test Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 5: Access Your Backend API Securely

🌐 https://api.learningtech.store

<img width="1512" height="936" alt="Screenshot 2026-02-01 at 11 41 06 PM" src="https://github.com/user-attachments/assets/3f6ff1d3-c5e3-4713-9a3a-126ff14394f7" />

If your backend api loads with a 🔒 lock icon, your SSL setup is successful! 🎉

---

# Creating Multiple Instances of Frontend Servers

This section explains how to scale the Travel Memory application by creating multiple EC2 instances using an AWS Launch Template.

---

## Step 1: Create a Launch Template from an Existing EC2 Instance

1. Navigate to your existing EC2 instance created for the Travel Memory application.
2. Click:  
   **Actions → Images & Templates → Create template from instance**
3. Enter the template name:  
   `travel-memory-frontend`
   `travel-memory-frontend-template`
5. Click **Create launch template**.

---

## Step 2: Launch Instances from the Template

1. Go to:  
   **Actions → Launch instance from template**
2. Select the source template:  
   `travel-memory-frontend`
3. Click **Launch instance**.

---

## Step 3: Verify Instances

1. In the AWS Console, confirm that two or more instances are now running under the same template.
2. Example access URL:  

```
http://<EC2_PUBLIC_IP>
```

---

You have successfully created multiple instances for your Travel Memory application!

# Creating and Attaching a Load Balancer to EC2 Instances

This guide explains how to create and attach an **Application Load Balancer (ALB)** to the EC2 instances running the **Travel Memory** application.

---

## Step 1: Configure the Load Balancer

1. Navigate to: EC2 → Load Balancers
2. Click **Create Load Balancer**.
3. Select **Application Load Balancer (ALB)**.
4. Set the load balancer name: travel-memory-frontend-lb
5. Set the scheme: internet-facing
6. Set the IP address type: IPv4
7. Under **Availability Zones**, select the same AZs as your EC2 instances.

---

## Step 2: Create a Target Group

1. Under **Listeners and routing**, click **Create a target group**.
2. Choose: Target type: Instances
3. Set: Protocol: HTTPS Port: 443

4. Select the two EC2 instances running the Travel Memory application.
5. Click **Include as pending**.
6. Click **Create target group**.

>  HTTPS should be terminated at the Load Balancer using ACM, while backend traffic remains HTTPS.

---

## Step 3: Create the Load Balancer

1. Return to **Load Balancers**.
2. Attach the target group you created.
3. Click **Create load balancer**.

---

## Step 4: Test the Load Balancer

1. Copy the DNS name of the load balancer, for example: travel-memory-frontend-lb-2044805926.ap-south-1.elb.amazonaws.com

2. Paste it into your browser: http://travel-memory-frontend-lb-2044805926.ap-south-1.elb.amazonaws.com

3. Verify that the Travel Memory application loads successfully.

---

**Your Load Balancer is now correctly distributing traffic across your EC2 instances!**

# AWS Certificate Manager (ACM) – SSL Certificate Details

This document records the issued SSL certificate used for securing the Travel Memory application.

---

## Certificate Information

- **Certificate ID:**  
  `cc3667a6-3da5-4921-84f9-fd6293be7300`

- **ARN:**  
  `arn:aws:acm:ap-south-1:233245302554:certificate/cc3667a6-3da5-4921-84f9-fd6293be7300`

- **Type:** Amazon Issued  
- **Region:** Asia Pacific (Mumbai)  
- **Account:** AvinashSain (233245302554)  
- **Status:** ✅ Issued  
- **In Use:** Yes  

---

## 🌐 Domain Secured

| Domain               | Status   |
|----------------------|----------|
| lb.learningtech.store | Success  |

---

🌐 https://lb.learningtech.store

<img width="1512" height="890" alt="Screenshot 2026-02-01 at 11 41 20 PM" src="https://github.com/user-attachments/assets/48e44e34-4eac-41c6-9ec6-ebee1916d544" />


## Renewal

- **Renewal Eligibility:** Eligible  
- **Renewal Status:** Automatic (AWS-managed)

---

## Validation Method

- **Type:** DNS (CNAME)
- **Record Name:**  
  `_da979cbccb069db6bbf69dbaf1308772.lb.learningtech.store`

---

## Applying Certificate to Load Balancer

1. Go to **EC2 → Load Balancers**.
2. Select your ALB.
3. Edit **Listeners**.
4. Add or modify: HTTPS: 443 → Forward to target group

5. Select this ACM certificate.

---

🎉 **Your domain is now secured with HTTPS using AWS ACM!**

# Auto Scaling Group (ASG) Setup for Travel Memory Application

This document describes the Auto Scaling Group configuration used to scale the Travel Memory frontend instances automatically.

---

## Auto Scaling Group Details

- **Auto Scaling Group Name:**  
  `travel-memory-frontend-asg`

- **Region:**  
  Asia Pacific (Mumbai)

- **Account:**  
  AvinashSain (233245302554)

- **Launch Template Used:**  
  `travel-memory-frontend-template`

- **Last Updated:**  
  1 minute ago

---

## Capacity Settings

- **Desired Capacity:** 1  
- **Minimum Capacity:** 1  
- **Maximum Capacity:** 2 *(recommended for scaling)*

---

## Instance Status

- Instances are launched automatically based on the launch template.
- Health checks ensure only healthy instances receive traffic from the Load Balancer.

---

## Integration

- The ASG is attached to the Application Load Balancer target group: travel-memory-frontend-tg

---

## Benefits

- Automatic instance replacement if one fails.
- Automatic scaling based on traffic load.
- Zero-downtime deployments when combined with rolling updates.

---

🎉 **Your Travel Memory application is now fully scalable and highly available!**

## Running Apps with PM2

### Install PM2

```bash
sudo npm install -g pm2
```

---

### Backend with PM2

```bash
cd ~/TravelMemory/backend
pm2 start index.js --name "travel-backend"
pm2 startup
pm2 save
```

---

### Frontend with PM2

```bash
cd ~/TravelMemory/frontend
npm install
npm run build
sudo npm install -g serve
pm2 start serve --name "travel-frontend" -- -s build -l 3000
pm2 save
```

---

### Verify

```bash
pm2 list
```

Expected:

```
travel-backend
travel-frontend
```

---

## Deployment Complete!

Your **Travel Memory** application is now:

- ✅ Secure (HTTPS)
- ✅ Scalable (ASG + ALB)
- ✅ Highly Available
- ✅ Production-ready

---

## Author

**Avinash Sain**  
GitHub: https://github.com/Avinashsain
