Internet
   ↓
Internet Gateway
   ↓
Route Table
   ↓
NACL
   ↓
Subnet
   ↓
Security Group
   ↓
EC2

# AWS VPC — Quick Revision with Architecture Diagram

## Core Concepts

| Concept | Purpose |
|---|---|
| **VPC** | Private network in AWS |
| **CIDR** | Defines IP address range |
| **Subnet** | Divides the VPC network |
| **Public Subnet** | Has route to Internet Gateway |
| **Private Subnet** | No direct route to Internet Gateway |
| **Route Table** | Controls traffic routing |
| **IGW** | Connects VPC to internet |
| **NAT Gateway** | Outbound internet for private subnet |
| **Security Group** | Resource/ENI-level firewall |
| **NACL** | Subnet-level firewall |

## Architecture Diagram

```text
                         INTERNET
                            │
                            ▼
                   ┌─────────────────┐
                   │ Internet Gateway│
                   │      (IGW)      │
                   └────────┬────────┘
                            │
          ┌─────────────────────────────────┐
          │        VPC 10.0.0.0/16          │
          │                                 │
          │   ┌─────────────────────────┐   │
          │   │ Public Subnet           │   │
          │   │ 10.0.1.0/24            │   │
          │   │                         │   │
          │   │ NACL                    │   │
          │   │   ↓                     │   │
          │   │ Security Group          │   │
          │   │   ↓                     │   │
          │   │ EC2                     │   │
          │   │                         │   │
          │   │ NAT Gateway             │   │
          │   └──────────┬──────────────┘   │
          │              │                  │
          │              ▼                  │
          │   ┌─────────────────────────┐   │
          │   │ Private Subnet          │   │
          │   │ 10.0.2.0/24            │   │
          │   │                         │   │
          │   │ NACL                    │   │
          │   │   ↓                     │   │
          │   │ Security Group          │   │
          │   │   ↓                     │   │
          │   │ EC2                     │   │
          │   └─────────────────────────┘   │
          │                                 │
          └─────────────────────────────────┘
```

## Public Subnet Flow

```text
Internet
   ↕
IGW
   ↕
Route Table
   ↕
NACL
   ↕
Security Group
   ↕
EC2
```

## Private Subnet Internet Flow

```text
Private EC2
    ↓
Security Group
    ↓
NACL
    ↓
Route Table
    ↓
NAT Gateway
    ↓
IGW
    ↓
Internet
```

## SG vs NACL

| Security Group | NACL |
|---|---|
| Resource/ENI level | Subnet level |
| **Stateful** | **Stateless** |
| Allow only | Allow + Deny |
| Return traffic automatic | Return traffic rule required |

This is a good structure for a **quick VPC revision before teaching Security Groups and NACLs**.

One technical correction: in the architecture diagram, the NAT Gateway should not appear as if it directly connects the public subnet to the private subnet. The **private subnet route table sends internet-bound traffic to a NAT Gateway located in the public subnet**, and the NAT Gateway reaches the IGW.

A more accurate core flow is:

```text
PUBLIC EC2
   ↕
Security Group
   ↕
NACL
   ↕
Public Subnet Route Table
   ↕
Internet Gateway
   ↕
Internet
```

```text
PRIVATE EC2
   ↓
Security Group
   ↓
NACL
   ↓
Private Subnet Route Table
   ↓
NAT Gateway (Public Subnet)
   ↓
Public Subnet Route Table
   ↓
Internet Gateway
   ↓
Internet
```

For your training, the key sequence to explain is:

**VPC → CIDR → Subnet → Route Table → IGW/NAT → NACL → Security Group → EC2**

Then move directly into **Security Groups first**, followed by **NACLs**.
