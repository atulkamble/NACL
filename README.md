# AWS NACL (Network Access Control List) Complete Learning, Practice & Interview Guide

---

# Module 1: Introduction to NACL

## What is NACL?

NACL (Network Access Control List) is a subnet-level virtual firewall that controls inbound and outbound traffic for one or more subnets within a VPC.

NACL acts as an additional security layer before traffic reaches the EC2 instance.

---

## Full Forms

| Term | Full Form                      |
| ---- | ------------------------------ |
| AWS  | Amazon Web Services            |
| VPC  | Virtual Private Cloud          |
| NACL | Network Access Control List    |
| SG   | Security Group                 |
| IGW  | Internet Gateway               |
| NAT  | Network Address Translation    |
| CIDR | Classless Inter-Domain Routing |
| EC2  | Elastic Compute Cloud          |
| ALB  | Application Load Balancer      |

---

# Module 2: Traffic Flow Understanding

## AWS Packet Flow

Internet User → Internet Gateway → Route Table → NACL → Security Group → EC2 Instance

Traffic must pass all layers.

If any layer blocks traffic, communication fails.

---

## AWS Network Security Layers

Layer 1 → Route Table

Layer 2 → Network ACL

Layer 3 → Security Group

Layer 4 → Operating System Firewall

Layer 5 → Application

---

# Module 3: Stateful vs Stateless

## NACL = Stateless

NACL does not remember previous requests.

Request and response must be allowed separately.

Example:

Laptop → EC2 (SSH Port 22)

EC2 → Laptop (Ephemeral Port)

Both directions require rules.

---

## Security Group = Stateful

Security Groups remember existing connections.

If request is allowed:

Laptop → EC2

Return traffic automatically allowed.

No separate response rule required.

---

## Memory Trick

NACL = No Memory = Stateless

Security Group = Smart Guard = Stateful

---

# Module 4: NACL Characteristics

| Feature            | NACL   |
| ------------------ | ------ |
| Level              | Subnet |
| Stateful           | No     |
| Supports Deny      | Yes    |
| Supports Allow     | Yes    |
| Rule Order Matters | Yes    |
| Inbound Rules      | Yes    |
| Outbound Rules     | Yes    |
| Applied To         | Subnet |

---

# Module 5: NACL vs Security Group

| Feature         | NACL       | Security Group |
| --------------- | ---------- | -------------- |
| Level           | Subnet     | Instance       |
| Stateful        | No         | Yes            |
| Supports Deny   | Yes        | No             |
| Rule Processing | Sequential | All Rules      |
| Return Traffic  | Manual     | Automatic      |
| Applied To      | Subnet     | ENI/Instance   |

---

# Module 6: NACL Rule Processing

Rules are evaluated from lowest number to highest number.

Example:

Rule 90 → DENY 203.0.113.10

Rule 100 → ALLOW 0.0.0.0/0

Result:

Traffic from 203.0.113.10 is denied because Rule 90 is processed first.

---

# Module 7: Default vs Custom NACL

## Default NACL

Inbound → Allow All

Outbound → Allow All

---

## Custom NACL

Inbound → Deny All

Outbound → Deny All

Until rules are added.

---

# Module 8: Common Ports

| Service    | Port |
| ---------- | ---- |
| SSH        | 22   |
| HTTP       | 80   |
| HTTPS      | 443  |
| RDP        | 3389 |
| DNS        | 53   |
| MySQL      | 3306 |
| PostgreSQL | 5432 |
| SQL Server | 1433 |
| Oracle     | 1521 |

---

# Module 9: Ephemeral Ports

Temporary ports used for return traffic.

AWS Recommendation:

1024-65535

Examples:

SSH Response

HTTP Response

HTTPS Response

RDP Response

Without Ephemeral Ports:

Connection fails.

---

# Module 10: NACL Architecture

VPC (10.0.0.0/16)

Public Subnet (10.0.1.0/24)

Private Subnet (10.0.2.0/24)

Web Server

Database Server

Public-NACL

Private-NACL

---

# Module 11: Scenario 1 – Public Web Server

Requirement:

Allow Website Access

Allow SSH Administration

Configuration:

Inbound

HTTP 80

HTTPS 443

SSH 22 (Admin IP Only)

Outbound

1024-65535

Result:

Website accessible.

SSH restricted.

---

# Module 12: Scenario 2 – Database Protection

Requirement:

Database accessible only from application subnet.

Configuration:

Inbound

Port 3306

Source:

10.0.1.0/24

Outbound

1024-65535

Result:

Internet users cannot directly access DB.

---

# Module 13: Scenario 3 – Block Malicious IP

Requirement:

Block attacker IP.

Configuration:

Rule 90

203.0.113.10/32

DENY

Rule 100

0.0.0.0/0

ALLOW

Result:

Only attacker blocked.

---

# Module 14: Scenario 4 – Block Entire Network

Requirement:

Block corporate network.

Configuration:

192.168.1.0/24

DENY

0.0.0.0/0

ALLOW

Result:

Entire network blocked.

---

# Module 15: Scenario 5 – Bastion Host

Requirement:

SSH only through Bastion Host.

Public Subnet:

Allow SSH from Admin IP.

Private Subnet:

Allow SSH only from Bastion Subnet.

Result:

Private servers protected.

---

# Module 16: AWS CLI Commands

## Create NACL

aws ec2 create-network-acl 
--vpc-id vpc-123456

---

## View NACLs

aws ec2 describe-network-acls

---

## Create Inbound Rule

aws ec2 create-network-acl-entry 
--network-acl-id acl-123456 
--rule-number 100 
--protocol 6 
--rule-action allow 
--egress false 
--cidr-block 0.0.0.0/0 
--port-range From=80,To=80

---

## Create Outbound Rule

aws ec2 create-network-acl-entry 
--network-acl-id acl-123456 
--rule-number 100 
--protocol 6 
--rule-action allow 
--egress true 
--cidr-block 0.0.0.0/0 
--port-range From=1024,To=65535

---

## Delete Rule

aws ec2 delete-network-acl-entry 
--network-acl-id acl-123456 
--rule-number 100 
--egress false

---

## Associate NACL

aws ec2 replace-network-acl-association 
--association-id aclassoc-123456 
--network-acl-id acl-123456

---

# Module 17: Troubleshooting Checklist

## SSH Not Working

Check:

EC2 Running

Public IP

Security Group 22

NACL 22

Ephemeral Ports

Route Table

Internet Gateway

---

## Website Not Opening

Check:

Apache/Nginx Running

Security Group

NACL

Port 80

Port 443

---

## Database Not Reachable

Check:

Database Listening

Port 3306

Security Group

NACL

Subnet Routing

---

# Module 18: Production Best Practices

Do not use Allow All.

Use least privilege.

Restrict SSH to office IP.

Restrict RDP to office IP.

Use NACL Deny Rules for known attackers.

Separate Public and Private NACLs.

Document all rule numbers.

Keep gaps between rules.

Example:

100

110

120

130

Allows future expansion.

---

# Module 19: Exam Questions

1. What is NACL?

2. Why is NACL stateless?

3. Difference between Security Group and NACL?

4. Which AWS firewall supports Deny rules?

5. Why are Ephemeral Ports required?

6. Can one NACL be attached to multiple subnets?

7. What happens if no rule matches?

8. Which rule is processed first?

9. Default NACL vs Custom NACL?

10. Why is NACL considered subnet-level security?

---

# Module 20: Important Points to Remember

✓ NACL works at Subnet Level

✓ NACL is Stateless

✓ Supports Allow and Deny

✓ Lowest Rule Number Wins

✓ One Subnet can have only One NACL

✓ One NACL can be attached to Multiple Subnets

✓ Ephemeral Ports Required

✓ Default NACL Allows All

✓ Custom NACL Denies All

✓ Security Group is Stateful

✓ NACL is first line of defense at subnet level

---

# Golden Rule

Traffic Flow:

Route Table

↓

Network ACL

↓

Security Group

↓

Operating System Firewall

↓

Application

Successful communication requires all layers to permit traffic.
