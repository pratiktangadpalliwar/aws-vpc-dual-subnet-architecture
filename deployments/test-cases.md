# Test Cases: AWS VPC Public/Private Subnet with NAT Gateway

This document verifies the functionality of your AWS VPC architecture including:
- Internet access
- NAT routing
- Bastion access
- RDP connections

---

## 🔹 1. RDP into Public EC2 Instance

**Action**: Use Remote Desktop (RDP) from your local machine to connect to the public EC2 instance using its **public IP**.

**Expected Result**:  
- Successful connection  
- Windows login screen appears

---

## 🔹 2. Access Internet from Public EC2

**Action**: From inside the public EC2:
- Open a browser or run:
  ```
  ping google.com
  ```
**Expected Result**:
- Internet accessible from the public subnet via Internet Gateway

---
## 🔹 3. Connect to Private EC2 from Public EC2

**Action**:  
From inside the public EC2 (bastion), use RDP to connect to the private EC2 using its **private IP**

**Expected Result**:
- RDP login screen appears  
- Access granted via internal network

---

## 🔹 4. Access Internet from Private EC2

**Action**:  
From inside the private EC2 (accessed through the public EC2), open a browser or run:
```
ping google.com
```
**Expected Result**:
- Internet is accessible via NAT Gateway
- Private EC2 has no public IP, but can reach internet outbound

---

## 🔹 5. Block Direct Internet Access to Private EC2

**Action**:  
Try to RDP into the private EC2 directly from your local machine.

**Expected Result**:  
- Connection fails  
- Expected behavior — Private instance is **not publicly accessible**

---

## 🔹 6. Security Group Rule Validation

**Action**:  
- Check inbound rules for `<sg_name>`  
- Ensure **RDP (TCP 3389)** is allowed **only from your IP**

**Expected Result**:  
- Only your IP can RDP into **public EC2**  
- **Private EC2** only accepts internal connections (no direct inbound)

---

## 🔹 7. Route Table Association Check

**Action**: Confirm that:  
- Public subnet is associated with `public-rt` (`0.0.0.0/0 → IGW`)  
- Private subnet is associated with `private-rt` (`0.0.0.0/0 → NAT Gateway`)

**Expected Result**:  
- Correct route table association  
- Routes are active and functional

---

## Summary Table

| Test Case                                     | Status     |
|----------------------------------------------|------------|
| RDP into Public EC2                           | Pass    |
| Internet from Public EC2                      | Pass    |
| RDP from Public to Private EC2                | Pass    |
| Internet from Private EC2 via NAT             | Pass    |
| Direct access to Private EC2 from internet    | Blocked |
| Security Group configuration                  | Valid   |
| Route table logic                             | Valid   |
---