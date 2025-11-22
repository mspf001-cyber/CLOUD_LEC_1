# ☁️ AWS EC2 Web Server Deployment: From Launch to SSL

![AWS](https://img.shields.io/badge/Cloud-AWS-232F3E?style=for-the-badge&logo=amazon-aws)
![Linux](https://img.shields.io/badge/OS-Amazon_Linux_2023-F7931E?style=for-the-badge&logo=linux)
![Apache](https://img.shields.io/badge/Server-Apache_httpd-D22128?style=for-the-badge&logo=apache)
![SSL](https://img.shields.io/badge/Security-Let's_Encrypt-003A70?style=for-the-badge&logo=letsencrypt)

## 📖 Project Overview
This project documents the complete lifecycle of deploying a secure, static web application on the AWS Cloud. It demonstrates proficiency in **Cloud Infrastructure (EC2)**, **Linux System Administration**, **Networking (DNS/Route 53)**, and **Cybersecurity (SSL/TLS)**.

**Live Demo Domain:** `https://birjeshkh.dpdns.org` *(Note: Infrastructure may be terminated to save costs)*

---

## 📑 Table of Contents
1. [Phase 1: Infrastructure Setup](#phase-1-infrastructure-setup)
2. [Phase 2: Web Server Configuration](#phase-2-web-server-configuration)
3. [Phase 3: Domain Mapping (DNS)](#phase-3-domain-mapping-dns)
4. [Phase 4: SSL Security Implementation](#phase-4-ssl-security-implementation)
5. [Phase 5: Cost Optimization (Teardown)](#phase-5-cost-optimization-teardown)
6. [Phase 6: Troubleshooting & Lessons Learned](#phase-6-troubleshooting--lessons-learned)

---

## Phase 1: Infrastructure Setup

### 1.1 Instance Launch
![Instance Dashboard](Screenshot%202025-11-21%20104645.png)
* **Provider:** AWS EC2 (Elastic Compute Cloud)
* **Region:** `us-east-1` (N. Virginia)
* **AMI:** **Amazon Linux 2023** (The successor to Amazon Linux 2)
* **Instance Type:** `t3.micro` (Free Tier Eligible)
* **Key Pair:** Generated `.pem` file for SSH authentication.
### 1.2 SSH Access (Windows)
Since the default `.pem` format is incompatible with PuTTY on Windows, a conversion was required.
1.  **Tool:** PuTTYgen
2.  **Action:** Converted `key.pem` → `key.ppk`.
3.  **Login:** Used **PuTTY** to connect via port 22.
    * *User:* `ec2-user`
    * *Host:* [Public IP Address]

---

## Phase 2: Web Server Configuration

### 2.1 Installing Apache
Amazon Linux 2023 uses `dnf` as the package manager.

```bash
# Update system packages
sudo dnf update -y

# Install Apache HTTP Server
sudo dnf install httpd -y

# Start the service
sudo systemctl start httpd

# Enable auto-start on reboot
sudo systemctl enable httpd
``` 

### 2.2 Configuring the Firewall (Security Groups)
![Security Group Rules](Screenshot%202025-11-21%20121927.png)
Initially, the website was inaccessible. This was due to the default AWS Security Group rules blocking inbound web traffic.

Action Taken: Modified Inbound Rules in AWS Console.

SSH (22): Restricted to My IP (Admin access).

HTTP (80): Open to 0.0.0.0/0 (Public).

HTTPS (443): Open to 0.0.0.0/0 (Secure traffic - added in Phase 4).
### 2.3 Deploying Content
```bash
# Overwrite index.html
sudo sh -c 'echo "<b>any message</b>" > /var/www/html/index.html'
```
![HTTP Test Page](Screenshot%202025-11-21%20104533.png)
Result: The website became visible over HTTP.

## Phase 3: Domain Mapping (DNS)
### 3.1 Domain Registration
Registrar: DigitalPlat
![Registrar Dashboard](Screenshot%202025-11-21%20114709.png)
Domain: birjeshkh.dpdns.org

### 3.2 AWS Route 53 Integration
To utilize AWS's advanced DNS routing, I migrated the Nameservers.

Created a Hosted Zone in AWS Route 53.

Extracted the 4 AWS Nameservers (ns-xxx.awsdns...).

Updated the Registrar's configuration to point to these AWS servers.

### 3.3 Record Creation
![Route 53 Config](Screenshot%202025-11-21%20114428.png)
Created an A Record to map the domain to the server's IP.

Type: A

Value: 34.230.76.188 (EC2 Public IP)

TTL: 120 seconds (Short TTL allowed for rapid propagation testing).

Routing: Simple Routing.
> **Verification:** The domain `birjeshkh.dpdns.org` successfully resolves to the server, though it is not yet secure.
![Domain Propagation Test](Screenshot%202025-11-21%20114913.png)
## Phase 4: SSL Security Implementation
To resolve the "Not Secure" browser warning, I implemented HTTPS using Let's Encrypt.

### 4.1 Installation
```Bash
# Install Certbot and Apache plugin for AL2023
sudo dnf install -y certbot python3-certbot-apache
```
### 4.2 VirtualHost Configuration
Certbot initially failed because Apache didn't explicitly define the domain name. I manually configured the VirtualHost.

File: /etc/httpd/conf.d/birjesh_ssl.conf

```Apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ServerName birjeshkh.dpdns.org
    ErrorLog /var/log/httpd/error.log
    CustomLog /var/log/httpd/access.log combined
</VirtualHost>
```
### 4.3 Generating the certificate

```bash
sudo systemctl restart httpd
sudo certbot --apache
```
Selected the domain birjeshkh.dpdns.org.

Enabled automatic redirects (HTTP -> HTTPS).
![Secure HTTPS Site](Screenshot%202025-11-21%20122032.png)
Final Result: The website is now fully secured with a valid SSL certificate.

## Phase 5: Cost Optimization (Teardown)
To adhere to the "Student Budget" protocol ($0.00 billing), a strict teardown process was followed immediately after project completion.

🛑 The "Scorched Earth" Checklist
Terminate Instance:

EC2 Dashboard -> Instance State -> Terminate.

Verifies that compute charges stop immediately.

Delete Route 53 Hosted Zone:

Step A: Deleted all records inside the zone (A, CNAME) except NS and SOA.

Step B: Deleted the Hosted Zone itself.

Prevents the $0.50/month recurring fee.

Release Elastic IP:

Network & Security -> Elastic IPs -> Release Address.

Prevents hourly penalty for unused static IPs.

Cleanup EBS Volumes:

Verified that the root volume was deleted automatically upon instance termination.

## Phase 6: Troubleshooting & Lessons Learned

This project involved navigating several common pitfalls in cloud deployment. Below is a log of issues encountered and their solutions.

### 🔧 Troubleshooting Log

| Issue / Error Message | Root Cause | Solution |
| :--- | :--- | :--- |
| **"This site can't be reached"** (Timeout) | AWS Security Groups block all inbound traffic by default. | **Action:** Added an Inbound Rule for `HTTP` (Port 80) allowing Source `0.0.0.0/0`. |
| **"Unable to find a virtual host listening on port 80"** | Certbot failed because Apache did not have a specific `ServerName` configuration for the domain. | **Action:** Created a custom config file at `/etc/httpd/conf.d/birjesh_ssl.conf` defining the `<VirtualHost>`. |
| **HTTPS Connection Timeout** | After installing SSL, the site refused to load despite the certificate being valid. | **Action:** The Security Group allowed Port 80 but blocked Port 443. Added an Inbound Rule for `HTTPS` (Port 443). |
| **"The specified hosted zone contains non-required resource record sets"** | Attempted to delete the Route 53 Hosted Zone while it still contained custom records. | **Action:** Manually deleted the `A` records inside the zone *before* deleting the Hosted Zone itself. |
| **PuTTY "Connection Refused"** | Accidentally replaced the SSH rule (Port 22) with the HTTP rule (Port 80) instead of adding a new one. | **Action:** Modified Security Group to ensure **BOTH** Port 22 (SSH) and Port 80 (HTTP) were open. |

### 🧠 Key Lessons Learned

1.  **OS Differences Matter:** Amazon Linux 2023 uses `dnf` instead of `yum` or `apt`. Copy-pasting generic Linux commands (like for Ubuntu) will fail.
2.  **The "Firewall" is Outside:** Unlike local servers, AWS security is managed at the network level (Security Groups), not just inside the OS. Opening a port in Apache is useless if AWS blocks it.
3.  **Cost Management:** "Closing the window" does not stop billing. You must explicitly **Terminate** instances and **Delete** Route 53 zones to ensure a $0.00 bill.
4.  **DNS Propagation:** Setting a low **TTL** (Time To Live) like 60 seconds is crucial during testing. It allows DNS changes to reflect almost immediately, preventing long waits after making mistakes.
