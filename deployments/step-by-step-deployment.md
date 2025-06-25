# Step-by-Step Deployment: AWS VPC with Public and Private Subnets + NAT Gateway

This guide explains how to deploy a secure two-tier network using **Amazon VPC**, where:
- A **public subnet EC2 instance** is internet-accessible
- A **private subnet EC2 instance** accesses the internet securely through a **NAT Gateway**

---

## Prerequisites

- AWS account with admin permissions
- Key pair for EC2 login (Windows in this case)
- Basic understanding of VPCs and EC2 networking

---

## 🔹 Step 1: Create a VPC

1. Go to the **VPC Dashboard** → *Create VPC*
2. Name: `<name>`
3. IPv4 CIDR block: `<CIDR Range>`
4. Tenancy: Default
5. Click **Create VPC**

---

## 🔹 Step 2: Create and Attach Internet Gateway

1. Go to **Internet Gateways** → *Create internet gateway*
2. Name: `<name>`
3. Click **Create** and then **Attach to VPC**
4. Select your newly created VPC

---

## 🔹 Step 3: Create Subnets

### Public Subnet:
- Name: `<name>`
- CIDR block: `<CIDR Range>`
- AZ: `<az>`

### Private Subnet:
- Name: `<name>`
- CIDR block: `<CIDR Range>`
- AZ: `<az>`

> Ensure both subnets are in the same availability zone

---

## 🔹 Step 4: Create Route Tables

### Public Route Table:
1. Name: `<name>`
2. Associate with: `public-subnet`
3. Add route:  
   - Destination: `0.0.0.0/0`  
   - Target: Internet Gateway (IGW-VPC2368)

### Private Route Table:
1. Name: `<name>`
2. Associate with: `private-subnet`
3. Initially leave empty; will configure route after NAT Gateway is created

---

## 🔹 Step 5: Allocate an Elastic IP

1. Go to **Elastic IPs** → *Allocate new address*
2. Choose default settings
3. Note the IP for NAT Gateway

---

## 🔹 Step 6: Create NAT Gateway

1. Go to **NAT Gateways** → *Create NAT Gateway*
2. Subnet: `public-subnet`
3. Elastic IP: Attach the one just created
4. Name: `<name>`
5. Click **Create NAT Gateway**
6. Wait until status = `available`

---

## 🔹 Step 7: Configure Private Route Table

1. Go to **Route Tables** → `private-rt`
2. Add route:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway (`<nat-gateway>`)

---

## 🔹 Step 8: Create a Security Group

1. Go to **EC2 > Security Groups** → *Create*
2. Name: `<name>`
3. Inbound Rules:
   - RDP (TCP 3389) from your IP (for Windows instances)
   - HTTP (80), HTTPS (443) (optional)
4. Outbound Rules: Leave as default (All traffic allowed)

---

## 🔹 Step 9: Create EC2 Instances

### Public EC2:
- Subnet: `public-subnet`
- Auto-assign Public IP: **Enable**
- AMI: Windows Server 2019 (or 2022)
- Instance Type: t2.micro (Free tier)
- Security Group: `<security-group>`
- Key Pair: Select existing or create new

### Private EC2:
- Subnet: `private-subnet`
- Auto-assign Public IP: **Disable**
- All other settings same as above

> Wait for both EC2s to reach `running` state.

---

## 🔹 Step 10: Testing

1. Connect to **Public EC2** using RDP
2. From inside that instance, use RDP to connect to the **Private EC2**
   - Use private IP of private instance
3. Inside Private EC2, open browser or ping a public IP (e.g. `ping google.com`)
   - Internet access should work via NAT Gateway

---

## Final Architecture
```
Custom VPC (10.0.0.0/16)
├── public-subnet (10.0.1.0/24)
│ ├── EC2: Public Windows (with public IP)
│ └── Internet Gateway
│
├── private-subnet (10.0.2.0/24)
│ ├── EC2: Private Windows (no public IP)
│ └── NAT Gateway (via public subnet)
│
├── Route Tables
│ ├── public-rt: 0.0.0.0/0 → IGW
│ └── private-rt: 0.0.0.0/0 → NAT Gateway
│
└── Shared Security Group & NACL
```
---

## Outcome

- Internet access from public EC2
- Internet access from private EC2 via NAT
- RDP access to public EC2
- Private EC2 not accessible directly from internet
---