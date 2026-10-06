# AWS Security Group — Core Guide

## 1. What is a Security Group?

A **Security Group (SG)** is a **stateful virtual firewall** that controls inbound and outbound traffic for AWS resources such as EC2.

```text
Internet
   │
   ▼
Security Group
   │
   ▼
EC2
```

## 2. Core Features

| Feature | Security Group |
|---|---|
| Level | ENI / Resource |
| Type | Stateful |
| Allow Rules | Yes |
| Deny Rules | No |
| Inbound | Yes |
| Outbound | Yes |
| Rule Priority | No |
| Return Traffic | Automatically allowed for established connections |

## 3. Stateful Concept

If SSH is allowed inbound:

```text
Laptop ───── TCP 22 ─────▶ EC2
Laptop ◀──── Response ───── EC2
```

The response traffic is automatically allowed.

**Security Group = Stateful**

## 4. Common Inbound Rules

| Service | Port | Source |
|---|---:|---|
| SSH | 22 | Admin-IP/32 |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |
| RDP | 3389 | Admin-IP/32 |
| MySQL | 3306 | Application-SG |

## 5. Security Group vs NACL

| Feature | Security Group | NACL |
|---|---|---|
| Level | Resource/ENI | Subnet |
| Stateful | Yes | No |
| Allow | Yes | Yes |
| Deny | No | Yes |
| Rule Priority | No | Lowest number first |
| Return Traffic | Automatic | Explicit rules required |

## 6. Create Security Group

```bash
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web Security Group" \
  --vpc-id vpc-xxxxxxxx
```

## 7. Allow SSH

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr YOUR-IP/32
```

## 8. Allow HTTP

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

## 9. Allow HTTPS

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

## 10. SG-to-SG Communication

Example: Web Server → Database

```text
Internet
   │
 80/443
   ▼
 Web-SG
   │
 Web EC2
   │
 3306
   ▼
 DB-SG
   │
 MySQL
```

DB-SG rule:

| Port | Source |
|---:|---|
| 3306 | Web-SG |

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-DB \
  --protocol tcp \
  --port 3306 \
  --source-group sg-WEB
```

## Key Points to Remember

**Security Group =**

```text
Resource / ENI Level
        +
Stateful
        +
ALLOW Rules Only
        +
Automatic Return Traffic
```

**NACL = Subnet Level**  
**Security Group = Resource Level**
