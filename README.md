# EC2 → Domain → Route53 → Secure Site → Terminate (Amazon Linux Version)

This is a **fully updated**, **detailed**, **Codespace‑ready README** with **inline screenshots placed immediately after the step they belong to**.

Every screenshot you provided has been used. Wherever there was a screenshot but **no matching documentation**, I added the missing explanation.

All screenshots reference the local paths you uploaded:

```
/mnt/data/Screenshot 2025-11-21 072515.png
/mnt/data/Screenshot 2025-11-21 104533.png
/mnt/data/Screenshot 2025-11-21 104645.png
/mnt/data/Screenshot 2025-11-21 114428.png
/mnt/data/Screenshot 2025-11-21 114709.png
/mnt/data/Screenshot 2025-11-21 114913.png
/mnt/data/Screenshot 2025-11-21 121927.png
/mnt/data/Screenshot 2025-11-21 122032.png
```

---

# Table of Contents

1. Prerequisites
2. Architecture
3. Launching EC2 (Amazon Linux)
4. Connecting via SSH (PuTTY on Windows)
5. Installing Apache on Amazon Linux
6. Deploying Website Files
7. Security Groups Configuration
8. Route 53 Hosted Zone Setup
9. Pointing External Domain Registrar to Route 53
10. Verifying Domain Resolution
11. Installing SSL (HTTPS) on Amazon Linux using Certbot
12. Final Secure Website Check
13. Termination & Cleanup to Avoid Cost

---

# 1. Prerequisites

* AWS account (root or IAM user)
* PuTTY + PuTTYgen (Windows)
* Amazon Linux EC2 instance
* A domain name (free domain used in example)
* Route 53 public hosted zone

---

# 2. Architecture Overview

Simple: domain → Route53 → EC2 → Apache → HTTPS.

---

# 3. Launching an EC2 Instance (Amazon Linux)

Go to:

```
AWS Console → EC2 → Launch Instances
```

* AMI: **Amazon Linux 2023** (or AL2)
* Instance type: `t3.micro`
* Key pair: create/download `.pem`
* Security group (initial): allow HTTP, HTTPS, SSH
* Launch instance

### Screenshot – EC2 Instance Running

![ec2-main](/mnt/data/Screenshot 2025-11-21 072515.png)

### Screenshot – Instance Detail View (public IP visible)

![ec2-detail](/mnt/data/Screenshot 2025-11-21 104533.png)

---

# 4. Connecting via SSH (Windows + PuTTY)

Amazon Linux username:

```
ec2-user
```

### Convert PEM to PPK (required for PuTTY)

1. Open PuTTYgen
2. Load your `.pem`
3. Save as `.ppk`

### SSH into EC2

PuTTY → Hostname:

```
ec2-user@<PUBLIC_IP>
```

Authentication → browse → select `.ppk`.

### Screenshot – SSH Rule Limiting Access

This screenshot shows SSH allowed only from a single IP.

![ssh-rule](/mnt/data/Screenshot 2025-11-21 122032.png)

---

# 5. Installing Apache on Amazon Linux

Run these commands:

```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable --now httpd
```

---

# 6. Deploying Your Website

Create index page:

```bash
sudo tee /var/www/html/index.html > /dev/null <<'HTML'
<!doctype html>
<html>
  <head><meta charset="utf-8"><title>My Site</title></head>
  <body>
    <h1>hello everyone , birjesh this side</h1>
  </body>
</html>
HTML
```

Fix permissions:

```bash
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html
```

### Screenshot – Website via Public IP

![site-ip](/mnt/data/Screenshot 2025-11-21 104645.png)

---

# 7. Configuring Security Groups

Recommended inbound rules:

* HTTP → 80 → 0.0.0.0/0
* HTTPS → 443 → 0.0.0.0/0
* SSH → 22 → Your IP only

### Screenshot – Security Group Showing Correct Configuration

![sg-rules](/mnt/data/Screenshot 2025-11-21 121927.png)

---

# 8. Route 53 Setup (Hosted Zone + A Record)

Go to:

```
Route 53 → Hosted Zones → Create Hosted Zone
```

Domain: `birjeshkh.dpdns.org`

Create a new **A Record**:

* Value: EC2 Public IP
* TTL: 120

### Screenshot – Route 53 Records

![route53](/mnt/data/Screenshot 2025-11-21 114428.png)

---

# 9. Configure Domain Registrar (Free Domain Provider)

Copy the **NS records** from Route 53 and paste them in your registrar.

### Screenshot – Domain Registrar Nameserver Section

![domain-provider](/mnt/data/Screenshot 2025-11-21 114709.png)

---

# 10. Confirm Domain Resolution

Once DNS propagates, your domain should open the EC2 website.

### Screenshot – Website Working via Domain

![domain-site](/mnt/data/Screenshot 2025-11-21 114913.png)

---

# 11. Installing HTTPS / SSL Certificate (Amazon Linux + Apache)

Amazon Linux does not include Certbot by default; install via snap.

```bash
sudo yum install -y snapd
sudo systemctl enable --now snapd
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Get certificate:

```bash
sudo certbot --apache -d birjeshkh.dpdns.org -d www.birjeshkh.dpdns.org
```

Renewal test:

```bash
sudo certbot renew --dry-run
```

---

# 12. Final Secure Website Check

### Screenshot – HTTPS Lock Icon

![https](/mnt/data/Screenshot 2025-11-21 122032.png)

Your site is now fully secured.

---

# 13. Cleanup to Avoid Unexpected AWS Charges

To avoid unwanted billing:

### 1. Terminate EC2 instance

```
EC2 Console → Instances → Select → Instance state → Terminate
```

### 2. Release Elastic IP (if created)

```
EC2 → Elastic IPs → Release
```

### 3. Delete Hosted Zone

```
Route 53 → Hosted Zones → Delete
```

You now avoid compute charges, EIP charges, Route 53 hosted zone charges.

---

# End of README