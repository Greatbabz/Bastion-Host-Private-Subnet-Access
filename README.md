# Week 3 · Day 4 | Bastion Host & Private Subnet Access

## Lab Objective

Launch an EC2 bastion host in a public subnet, SSH into it, and from there connect to a private subnet EC2 instance — demonstrating the bastion pattern used in production.

## Key Learning

> **A bastion host acts as a "jump server" — the only entry point to private instances. You SSH into the public bastion, then hop to private instances. No direct internet access to private resources.**

## Architecture Overview

### Connection Chain
[Your Windows PC]
│
│ PuTTY → SSH to Bastion (Public IP)
│ using bastion-key.ppk
▼
[Bastion-Host] (10.0.1.xxx) - Public Subnet ✅
│
│ ssh -i bastion-key.pem → 10.0.10.xxx
▼
[Private-App-Server] (10.0.10.xxx) - Private Subnet ✅
│
│ NO PUBLIC IP - Cannot be reached directly from internet 🔒

## What I Did

| Step | Action |
|------|--------|
| 1 | Launched Bastion Host in Public Subnet (Auto-assign Public IP: ENABLED) |
| 2 | Launched Private App Server in Private Subnet (Auto-assign Public IP: DISABLED) |
| 3 | Used SAME key pair for both instances (bastion-key) |
| 4 | Configured Security Group: Private instance allows SSH ONLY from Bastion SG |
| 5 | SSHed into Bastion Host using PuTTY |
| 6 | Copied the key to Bastion using pscp |
| 7 | SSH hopped from Bastion to Private instance |
| 8 | Verified connection with hostname and whoami |
| 9 | Terminated both instances after lab (cleanup) |

## AWS Resources Created

| Resource | Name | Details |
|----------|------|---------|
| Bastion Host | Bastion-Host | Public Subnet, Public IP assigned |
| Private Server | Private-App-Server | Private Subnet, Private IP: 10.0.10.xxx |
| Key Pair | bastion-key | RSA, .pem and .ppk formats |
| Security Group (Bastion) | SG-WebServers | SSH (port 22) from my IP |
| Security Group (Private) | SG-Private-App | SSH (port 22) from Bastion SG only |

## Security Group Configuration

| Security Group | Inbound Rule | Source | Purpose |
|----------------|--------------|--------|---------|
| **SG-WebServers (Bastion)** | SSH (port 22) | My Public IP | Only I can SSH into bastion |
| **SG-Private-App** | SSH (port 22) | SG-WebServers (Bastion) | Only bastion can reach private instance |

> **Critical:** Private instance security group references the Bastion's security group by ID, not by IP. This is more secure and dynamic.

## Verification Commands

### On Local Machine (Copy key to Bastion)On Bastion Host (Set permissions and connect)

```bash
pscp -i bastion-key.ppk bastion-key.pem ec2-user@[BASTION-PUBLIC-IP]:/home/ec2-user/

**On Bastion Host (Set permissions and connect)**
chmod 400 bastion-key.pem
ssh -i bastion-key.pem ec2-user@10.0.10.xxx

**On Private Instance (Verify)**
hostname
whoami

## Screenshots

Screenshot	Description
Instances Running	Bastion has public IP, Private has NO public IP
SSH to Bastion	Successfully connected to Bastion Host
SSH Hop to Private	SSH from Bastion into private instance
Instances Terminated	Cleanup - both instances terminated

## Key Takeaways

Bastion pattern = Single entry point to private subnets

Security Groups reference other SGs - More secure than IP-based rules

Same key pair needed for the hop

Private instances = NO public IPs - Completely isolated from internet

Cleanup is professional - Always terminate lab resources


