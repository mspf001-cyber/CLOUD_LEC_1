# CLOUD_LEC_1

Hosting a Website on AWS EC2 With Domain, Name Servers, and SSL

A complete step-by-step guide for deploying a website on EC2, configuring Route 53, updating name servers, and installing SSL using Let’s Encrypt.


---

1. Launch an EC2 Server

Steps

1. Sign in to AWS Console.


2. Open EC2 → Instances → Launch Instance.


3. Enter an instance name.


4. AMI: Select Amazon Linux 2 or Ubuntu.


5. Instance Type: Select t2.micro (Free Tier).


6. Key Pair: Create/download your key.


7. Network settings:

Use default VPC

Enable Auto-assign Public IP



8. Security Group rules:

Allow SSH (22)

Allow HTTP (80)

Allow HTTPS (443)



9. Click Launch Instance.




---

2. Connect to EC2 (SSH / PuTTY)

Linux/Mac Terminal

chmod 400 yourkey.pem
ssh -i "yourkey.pem" ec2-user@PUBLIC-IP

Windows Using PuTTY

1. Convert .pem → .ppk using PuTTYgen.


2. Open PuTTY → enter EC2 Public IP.


3. Add .ppk under SSH → Auth.


4. Connect and log in as:

ec2-user (Amazon Linux)

ubuntu (Ubuntu)





---

3. Install Apache or Nginx

Apache (Amazon Linux)

sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

Deploy Your Website Files

cd /var/www/html
sudo nano index.html

Add your HTML and save.


---

4. Create Hosted Zone in Route 53

1. Open Route 53 → Hosted Zones.


2. Click Create Hosted Zone.


3. Enter your domain name.


4. Copy the NS (Nameservers) that AWS provides.




---

5. Update Domain Registrar Name Servers

1. Open your domain provider’s dashboard (GoDaddy, Namecheap, Hostinger, etc.).


2. Go to DNS / Nameserver Settings.


3. Replace their NS records with the 4 NS values from Route 53.


4. Save changes.


5. Wait for DNS propagation (10 minutes → 24 hours).




---

6. Create DNS A Record in Route 53

In your Hosted Zone:

1. Click Create Record.


2. Select A – IPv4 Address.


3. Value = Your EC2 Public IP


4. Save.



Your domain now points to your EC2 instance.


---

7. Verify Website Before SSL

Open browser:

http://yourdomain.com

If the website loads, mapping is correct.

If it fails, verify:

A record correct

Nameservers updated

Port 80 open

Instance running

Apache active



---

8. Install SSL Certificate (Let’s Encrypt: Certbot)

Install Certbot (Amazon Linux)

sudo yum install -y certbot python3-certbot-apache

Run SSL Setup

sudo certbot --apache

Certbot will:

Detect your domain

Install SSL

Configure HTTPS redirection


Auto-Renewal

sudo crontab -e

Add:

0 3 * * * certbot renew --quiet


---
