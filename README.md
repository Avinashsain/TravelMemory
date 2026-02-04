
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
        proxy_pass http://127.0.0.1:3000;
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
        proxy_pass http://127.0.0.1:3001;
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
        proxy_pass http://127.0.0.1:3000;
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
        proxy_pass http://127.0.0.1:3001;
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

- Backend API:  
  ```text
  http://api.learningtech.store
  ```

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
        proxy_pass http://127.0.0.1:3000;
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

## 🔧 Final Nginx Configuration After SSL

```nginx
server {
    server_name api.learningtech.store;

    location / {
        proxy_pass http://127.0.0.1:3001;
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

If your backend api loads with a 🔒 lock icon, your SSL setup is successful! 🎉
