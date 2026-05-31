# AWS EC2 + Nginx + Cloudflare Tunnel Lab

## Project Overview

This project demonstrates how to deploy a web application on AWS using Amazon EC2 and Nginx while publishing the application through Cloudflare Tunnel and Cloudflare Zero Trust.

The goal was to:

* Build AWS networking from scratch
* Deploy an EC2 web server
* Manage the server using AWS Systems Manager (SSM) instead of SSH
* Configure Nginx as a web server
* Publish the website through Cloudflare Tunnel
* Understand the difference between public and private AWS architectures
* Troubleshoot real-world cloud networking issues

---

## Solution Architecture

![AWS EC2 + Cloudflare Tunnel Architecture](Screenshots/01-solution-architecture-diagram.png)

# Architecture Overview

Final Lab Architecture

```
                Internet User
                       │
                       ▼
             web.basitcloudlab.com
                       │
                       ▼
                Cloudflare DNS
                       │
                       ▼
             Cloudflare Edge Network
                       │
                       ▼
             Cloudflare Tunnel
                       │
             Encrypted Connection
                       │
                       ▼
             cloudflared Service
                       │
                       ▼
                localhost:80
                       │
                       ▼
                     Nginx
                       │
                       ▼
               Amazon Linux EC2
                       │
                       ▼
                 AWS VPC Network
```

---

# AWS Infrastructure Built

## VPC

Created a custom Virtual Private Cloud (VPC):

VPC Name:

basit-cloud-vpc

Purpose:

* Isolate resources from the default AWS network
* Create a dedicated lab environment

---

## Public Subnet

Created:

public-subnet-1

CIDR:

10.0.1.0/24

Purpose:

* Host EC2 resources
* Provide routing through an Internet Gateway

---

## Internet Gateway

Created:

basit-cloud-igw

Purpose:

* Allow traffic between the VPC and the Internet

Attached to:

basit-cloud-vpc

---

## Route Table

Created:

basit-cloud-rt

Routes:

10.0.0.0/16 → local

0.0.0.0/0 → Internet Gateway

Purpose:

* Route VPC traffic internally
* Provide internet routing

---

## Security Group

Created:

private-web-sg

Purpose:

* Control EC2 inbound and outbound traffic

Temporary Rules Used:

HTTP (80)

for Nginx testing

---

# EC2 Deployment

Created:

cloudflare-private-web

Configuration:

Operating System:
Amazon Linux 2023

Instance Type:
t2.micro

Management:
AWS Systems Manager Session Manager

Purpose:

* Host the web application
* Avoid using traditional SSH access

---

# IAM Role

Created:

EC2-SSM-Role

Attached Policy:

AmazonSSMManagedInstanceCore

Purpose:

* Allow Session Manager access
* Manage EC2 without SSH keys

---

# AWS Systems Manager (SSM)

Instead of:

Laptop
↓
SSH
↓
EC2

We used:

AWS Console
↓
Session Manager
↓
EC2

Benefits:

* No SSH keys
* No Port 22
* No Bastion Host
* Centralized management

---

# VPC Endpoints

Created:

* ssm
* ssmmessages
* ec2messages

Purpose:

Allow Systems Manager communication from EC2.

This demonstrated how AWS services communicate privately through VPC Endpoints.

---

# Nginx Web Server

Installed:

nginx

Commands:

sudo dnf install nginx -y

sudo systemctl enable nginx

sudo systemctl start nginx

Purpose:

Serve a custom web page.

Custom Page:

Basit Cloud Lab

Private AWS EC2 behind Cloudflare Tunnel

Built by Syed Basit Aftab

---

# Cloudflare Zero Trust

Activated:

Cloudflare Zero Trust Free

Purpose:

Provide secure access to internal resources.

Features Used:

* Cloudflare Tunnel
* Hostname Routing
* DNS Integration

---

# Cloudflare Tunnel

Created:

basit-cloud-tunnel

Purpose:

Securely expose the web application through Cloudflare.

Tunnel Flow:

Internet User
↓
Cloudflare
↓
Tunnel
↓
cloudflared
↓
localhost:80
↓
nginx

The cloudflared service running on EC2 maintained an outbound encrypted connection to Cloudflare.

No inbound tunnel connections were required.

---

# cloudflared Connector

Installed:

cloudflared

Purpose:

Create a secure outbound connection between EC2 and Cloudflare.

Tunnel Status:

Connected

Healthy

---

# Hostname Route

Created:

web.basitcloudlab.com

Mapped To:

http://localhost:80

Purpose:

Forward requests received by Cloudflare to the local Nginx service.

---

# DNS Configuration

Created:

web.basitcloudlab.com

Record Type:

Tunnel

Important:

No EC2 IP address was exposed in DNS.

Instead:

web.basitcloudlab.com
↓
Cloudflare Tunnel
↓
EC2

---

# Traffic Flow Explained

When a user visits:

https://web.basitcloudlab.com

Traffic follows:

Browser
↓
Cloudflare DNS
↓
Cloudflare Edge
↓
Cloudflare Tunnel
↓
cloudflared
↓
localhost:80
↓
nginx

Response follows the same path back to the user.

---

# Cloudflare Tunnel Security Benefits

Without Tunnel:

User
↓
Public EC2 IP
↓
Nginx

With Tunnel:

User
↓
Cloudflare
↓
Tunnel
↓
Nginx

Benefits:

* Hides origin server details from DNS
* Removes direct user access through Cloudflare hostname
* Uses encrypted tunnel communication
* Integrates with Cloudflare Zero Trust

---

# Major Troubleshooting Performed

## Session Manager Initially Offline

Problem:

EC2 could not register with Systems Manager.

Actions:

* Verified IAM Role
* Verified Route Table
* Created SSM VPC Endpoints
* Verified Endpoint Security Groups

Result:

Session Manager successfully connected.

---

## Nginx Installation Failed

Problem:

sudo dnf install nginx -y stalled.

Root Cause:

Instance had:

* No Public IP
* No NAT Gateway

Result:

No outbound internet connectivity.

Resolution:

Assigned a temporary public IP.

Result:

Nginx installed successfully.

---

## Cloudflare Tunnel Error 1033

Problem:

Website became unavailable after stopping cloudflared.

Observed:

Error 1033

Cloudflare Tunnel Error

Root Cause:

Tunnel service was unavailable.

Resolution:

Restarted cloudflared.

Result:

Website restored.

---

## Public Subnet vs Public Instance

Most Important Lesson Learned

Public Subnet:

Route to Internet Gateway

Public Instance:

Requires Public IP or Elastic IP

Key Discovery:

Public Subnet ≠ Public Instance

An EC2 instance without:

* Public IP
* Elastic IP
* NAT Gateway

cannot reach the Internet even when placed in a public subnet.

---

# Lessons Learned

* How AWS networking components interact
* How Internet Gateways function
* Difference between Public and Private Connectivity
* How Session Manager replaces SSH
* How VPC Endpoints work
* How Cloudflare Tunnel operates
* How cloudflared establishes outbound tunnels
* DNS routing through Cloudflare
* Real-world troubleshooting methodology
* Security-focused cloud architecture design

---

# Skills Demonstrated

AWS

* VPC
* Subnets
* Route Tables
* Internet Gateway
* Security Groups
* EC2
* IAM
* Systems Manager
* VPC Endpoints

Linux

* Amazon Linux
* Nginx
* systemctl
* Package Management
* Networking
* Service Management

Cloudflare

* DNS
* Zero Trust
* Cloudflare Tunnel
* cloudflared
* Hostname Routing

---

# Resume Bullet

Built and secured a cloud-hosted web application using AWS EC2, Nginx, IAM Roles, Systems Manager, VPC Endpoints, Cloudflare Zero Trust, and Cloudflare Tunnel. Configured secure remote management without SSH, implemented DNS-based application publishing through Cloudflare, and performed troubleshooting involving internet routing, VPC networking, endpoint connectivity, and tunnel-based application delivery.
