
# 🚀 Travel Memory Application Deployment on AWS

This repository documents the complete deployment of the **Travel Memory** MERN-stack application using AWS cloud services, including high availability, SSL, auto scaling, and domain integration.

---

## 📌 Project Overview

The **Travel Memory** application is a full-stack MERN application that allows users to store and manage travel memories. This guide explains how to:

- Deploy frontend and backend on AWS EC2
- Connect to MongoDB Atlas
- Configure Nginx as a reverse proxy
- Enable HTTPS using Certbot & AWS ACM
- Scale using Launch Templates and Auto Scaling Groups
- Attach an Application Load Balancer
- Connect a custom domain via Cloudflare

---

## 🧱 Architecture Diagram

```
User → Cloudflare Domain → AWS Load Balancer → EC2 Instances → Nginx → Node.js App → MongoDB Atlas
```

---

## 🛠️ Tech Stack

- **Frontend:** React.js
- **Backend:** Node.js, Express.js
- **Database:** MongoDB Atlas
- **Cloud:** AWS EC2, ALB, ASG, ACM
- **Web Server:** Nginx
- **Process Manager:** PM2
- **Domain & DNS:** Cloudflare
- **SSL:** Let’s Encrypt (Certbot) + AWS ACM

---

## 🚀 Backend Deployment on AWS EC2

### Step 1: Launch EC2 Instance
- Launch an Ubuntu EC2 instance.
- Connect via SSH.

---

### Step 2: Install Required Packages

```bash
sudo apt update -y
sudo apt install nodejs -y
sudo apt install npm -y

nodejs -v
npm -v
```

---

### Step 3: Clone Backend Repository

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

Add:

```env
PORT=3001
MONGO_URI=ENTER_YOUR_MONGODB_CONNECTION_STRING
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

Backend runs at:

```
http://<EC2_PUBLIC_IP>:3001
```

---

## 🌐 MongoDB Atlas Setup & Compass Connection

### Step 1: Log in
- Visit: https://cloud.mongodb.com/
- Log in to your account.

---

### Step 2: Create Organization & Project

---

### Step 3: Create Cluster
- Name: `herocluster1`
- Select **M0 Free Plan**
- Click **Create Deployment**

---

### Step 4: Create Database User
- Go to **Database Access**
- Add new user (username + password)

---

### Step 5: Configure Network Access
- Add IP:
  ```
  0.0.0.0/0
  ```

> ⚠️ For production, restrict to trusted IPs.

---

### Step 6: Connect MongoDB Compass
- Click **Connect → Compass**
- Copy connection string.

Example:

```
mongodb+srv://<username>:<password>@cluster0.mongodb.net/?retryWrites=true&w=majority
```

---

### Step 7: Connect Using Compass
- Open MongoDB Compass
- Paste connection string
- Click **Connect**

---

## 🌍 Frontend Deployment on AWS EC2

### Step 1: Launch EC2 Instance
- Launch Ubuntu EC2
- Connect via SSH.

---

### Step 2: Install Node & NPM

```bash
sudo apt update -y
sudo apt install nodejs -y
sudo apt install npm -y

nodejs -v
npm -v
```

---

### Step 3: Clone Frontend Repository

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

Add:

```env
REACT_APP_BACKEND_URL=http://<EC2_PUBLIC_IP>:3001
```

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

Frontend runs at:

```
http://<EC2_PUBLIC_IP>:3000
```

---

## 🔁 Reverse Proxy Setup with Nginx

### Frontend (Port 3000 → Port 80)

```bash
sudo apt update -y
sudo apt install nginx -y
nano /etc/nginx/sites-available/default
```

Paste:

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

Reload:

```bash
nginx -t
systemctl reload nginx
```

Test:

```
http://<EC2_PUBLIC_IP>
```

---

### Backend (Port 3001 → Port 80)

```bash
nano /etc/nginx/sites-available/default
```

Paste:

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

Reload:

```bash
nginx -t
systemctl reload nginx
```

Test:

```
http://<EC2_PUBLIC_IP>/trip
```

---

## 🌐 Custom Domains with Nginx

### Domains Used
- Frontend: `learningtech.store`
- Backend: `api.learningtech.store`

### DNS Records

| Type | Name | Value |
|------|------|--------|
| A    | @    | EC2_PUBLIC_IP |
| A    | www  | EC2_PUBLIC_IP |
| A    | api  | EC2_PUBLIC_IP |

---

### Nginx Configuration

```nginx
# Frontend
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

# Backend
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

Reload:

```bash
nginx -t
systemctl reload nginx
```

---

## 🔒 Enable HTTPS with Certbot (Frontend)

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d learningtech.store -d www.learningtech.store
```

---

### Final HTTPS URLs

- 🌐 https://learningtech.store
- 🌐 https://www.learningtech.store

---

## 🔐 Enable HTTPS with Certbot (Backend)

```bash
sudo certbot --nginx -d api.learningtech.store
```

---

### Final Backend HTTPS URL

- 🌐 https://api.learningtech.store

---

## ⚖️ Load Balancer Setup (ALB)

### Step 1: Create ALB
- Name: `travel-memory-frontend-lb`
- Scheme: Internet-facing
- IP Type: IPv4
- Select same AZs as EC2

---

### Step 2: Create Target Group
- Target type: Instances
- Protocol: HTTPS
- Port: 443
- Register EC2 instances

---

### Step 3: Attach Target Group to ALB

---

### Step 4: Test Load Balancer

```
http://travel-memory-frontend-lb-xxxx.ap-south-1.elb.amazonaws.com
```

---

## 🔏 AWS Certificate Manager (ACM)

- **Certificate ARN:**  
  `arn:aws:acm:ap-south-1:233245302554:certificate/cc3667a6-3da5-4921-84f9-fd6293be7300`

- **Domain Secured:** `lb.learningtech.store`
- **Validation:** DNS (CNAME)
- **Status:** Issued & Active

---

## 📈 Auto Scaling Group (ASG)

- **ASG Name:** `travel-memory-frontend-asg`
- **Launch Template:** `travel-memory-frontend-template`
- **Min:** 1
- **Desired:** 1
- **Max:** 2

Benefits:
- Auto recovery
- Auto scaling
- Zero-downtime deployments

---

## ⚙️ Running Apps with PM2

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

## 🎉 Deployment Complete!

Your **Travel Memory** application is now:

- ✅ Secure (HTTPS)
- ✅ Scalable (ASG + ALB)
- ✅ Highly Available
- ✅ Production-ready

---

## 👨‍💻 Author

**Avinash Sain**  
AWS Cloud & DevOps Enthusiast  
GitHub: https://github.com/Avinashsain
