# SSL-Using-Let-s-Encrypt-on-EC2
# SSL Certificate Setup with Let’s Encrypt, Nginx, and AWS Load Balancer

This guide explains how to generate an SSL certificate using **Certbot** on an **Nginx EC2 instance**, import it into **AWS Certificate Manager (ACM)**, and attach it to an **AWS Load Balancer**.

---

## Architecture Overview

- Domain points to **AWS Load Balancer**
- SSL certificate generated on **EC2 (Nginx)** using Let’s Encrypt
- Certificate imported into **ACM**
- Load Balancer handles HTTPS traffic
- HTTP traffic is redirected to HTTPS

-----------------

## Step 1: Generate Certificate Files on EC2

This step uses **Certbot** on your Nginx EC2 instance to generate SSL certificate files.

### Prerequisites
- Nginx installed and running
- Domain DNS **A / CNAME record** must point to the **Load Balancer**
- Port **80** must be open (for Let’s Encrypt validation)

### Commands (Run on EC2)

#### Install Certbot Nginx Plugin
```bash
sudo dnf install -y python3-certbot-nginx
```

### Generate Certificate Files
```sudo certbot certonly --nginx -d aniljadhav.co.in```

### Locate Generated Files
```ls /etc/letsencrypt/live/aniljadhav.co.in/```

### Generated Files
- cert.pem → Certificate body
- privkey.pem → Private key
- chain.pem → Certificate chain
- fullchain.pem → (Do NOT use directly in ACM)

### Important Notes
- We use certonly because Load Balancer, not Nginx, will handle SSL
- Nginx is used only for domain validation
- DNS must be correct before running Certbot

-----------------

## Step 2: Import Certificate into AWS Certificate Manager (ACM)
- AWS Load Balancers can only use certificates stored in ACM.
- Open ACM Console
- Go to AWS Console → Certificate Manager
- Click Import a certificate

 ### Map Files to ACM Fields
Certificate body: ```sudo cat /etc/letsencrypt/live/aniljadhav.co.in/cert.pem```

Certificate private key: ```sudo cat /etc/letsencrypt/live/aniljadhav.co.in/privkey.pem```

Certificate chain (optional):```sudo cat /etc/letsencrypt/live/aniljadhav.co.in/chain.pem```

----
 ### Very Important

DO NOT paste fullchain.pem into a single field
ACM requires:
- cert.pem → Certificate body
- chain.pem → Certificate chain
Using fullchain.pem causes:
- "exactly 1 certificate required" error

Result
Certificate status becomes Issued
Certificate is now available for Load Balancer usage

----------------
## Step 3: Configure Load Balancer Listeners
 ### A. Add HTTPS (Port 443) Listener
- Go to EC2 → Load Balancers
- Select your Load Balancer
- Open Listeners tab

Click Add listener
- Listener Configuration
- Protocol: HTTPS
- Port: 443
- Default Action: Forward to your Nginx Target Group
- SSL Certificate: ( Select From ACM and Choose the imported certificate )
- Click Save
  
-----
 ### B. Redirect HTTP (Port 80) to HTTPS (Recommended)
- Select existing HTTP : 80 listener
- Click Edit default actions
- Change action to Redirect
- Redirect Settings
- Protocol: HTTPS
- Port: 443
- Status Code: HTTP_301 (Permanent)
- Save changes

----------------
## Final Validation
https://aniljadhav.co.in

- SSL lock should be visible
- HTTP traffic should auto-redirect to HTTPS
```
- **Common Issues & Fixes**
- ❌ Certbot DNS error
- Ensure dmain points to Load Balancer
- ❌ ACM import error
- Do not use fullchain.pem
❌ HTTPS not working

Check listener and target group health
```

**Summary**
```- Let’s Encrypt → EC2 (Certbot)
- SSL files → Imported into ACM
- ACM → Attached to Load Balancer
- LoadBalancer → Handles HTTPS securely
```

**Use this Fancy HTML for Ngnix to check output**
```
cd /usr/share/nginx/html

sudo git clone git@github.com:aniljadhavmca/SSL-Using-Let-s-Encrypt-on-EC2.git

Amazon Linux 2 / 2023: /usr/share/nginx/html
```
