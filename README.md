# AWS NACL

## 1. What is NACL?

**NACL (Network Access Control List)** is a **subnet-level firewall** in an AWS VPC.

It controls traffic **entering and leaving a subnet**.

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
┌──────────────────────┐
│       Subnet         │
│                      │
│        NACL          │
│          │           │
│          ▼           │
│   Security Group     │
│          │           │
│          ▼           │
│         EC2          │
└──────────────────────┘
```

---

# 2. Core NACL Features

| Feature | NACL |
|---|---|
| Works at | Subnet level |
| Type | Stateless |
| ALLOW rules | Yes |
| DENY rules | Yes |
| Inbound rules | Yes |
| Outbound rules | Yes |
| Rule processing | Lowest number first |

---

# 3. NACL is Stateless

NACL does **not remember connections**.

If incoming traffic is allowed, the required return traffic must also be allowed.

```text
Client                         EC2
  │                             │
  │────── HTTP :80 ────────────▶│
  │                             │
  │◀────── Response ────────────│
  │       Client Port           │
```

Example:

```text
Inbound
TCP 80 → ALLOW

Outbound
TCP 1024-65535 → ALLOW
```

**Remember:**

> NACL = Stateless = Check both directions.

---

# 4. NACL vs Security Group

| Feature | NACL | Security Group |
|---|---|---|
| Level | Subnet | ENI/Resource |
| Stateful | ❌ | ✅ |
| Allow | ✅ | ✅ |
| Deny | ✅ | ❌ Explicit deny |
| Rule order | Lowest first | No rule priority |
| Return traffic | Explicitly permit | Automatic for established connections |

### Easy Memory

```text
NACL → Subnet + Stateless + Allow/Deny
SG   → Resource + Stateful + Allow
```

---

# 5. Rule Processing

NACL evaluates rules from the **lowest rule number to highest**.

Example:

| Rule | Source | Action |
|---:|---|---|
| 90 | 203.0.113.10/32 | DENY |
| 100 | 0.0.0.0/0 | ALLOW |
| * | All | DENY |

```text
203.0.113.10
      │
      ▼
 Rule 90
    DENY
      ✕
```

Even though Rule 100 allows traffic, Rule 90 matches first.

> **First matching rule wins.**

---

# 6. Default vs Custom NACL

| Type | Initial Behavior |
|---|---|
| Default NACL | Allows traffic |
| Custom NACL | Denies traffic until ALLOW rules are created |

---

# 7. Simple Web Server Example

Requirement:

```text
Internet
   │
   │ HTTP :80
   ▼
 NACL
   │
   ▼
  EC2
```

### Inbound

| Rule | Port | Source | Action |
|---:|---:|---|---|
| 100 | 80 | 0.0.0.0/0 | ALLOW |
| * | ALL | ALL | DENY |

### Outbound

| Rule | Port | Destination | Action |
|---:|---:|---|---|
| 100 | 1024-65535 | 0.0.0.0/0 | ALLOW |
| * | ALL | ALL | DENY |

---

# 8. Create NACL — CLI

### Create

```bash
aws ec2 create-network-acl --vpc-id vpc-09683ff71eaaac232
```
```
aws ec2 create-network-acl --vpc-id vpc-09683ff71eaaac232 --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=new-NACL}]'
```
### View

```bash
aws ec2 describe-network-acls
```

### Allow Inbound HTTP rule — Port 80

```bash
aws ec2 create-network-acl-entry \
  --cli-input-json '{
    "NetworkAclId": "acl-00d1a744ffe03a7b5",
    "RuleNumber": 100,
    "Protocol": "6",
    "RuleAction": "allow",
    "Egress": false,
    "CidrBlock": "0.0.0.0/0",
    "PortRange": {
      "From": 80,
      "To": 80
    }
  }'
```

### Allow Outbound return traffic

```bash
aws ec2 create-network-acl-entry \
  --cli-input-json '{
    "NetworkAclId": "acl-00d1a744ffe03a7b5",
    "RuleNumber": 100,
    "Protocol": "6",
    "RuleAction": "allow",
    "Egress": true,
    "CidrBlock": "0.0.0.0/0",
    "PortRange": {
      "From": 1024,
      "To": 65535
    }
  }'
```

---

# 9. DENY Rule Example

Block one IP:

```text
203.0.113.10/32
```

```bash
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxxxxx \
  --rule-number 90 \
  --protocol -1 \
  --rule-action deny \
  --egress false \
  --cidr-block 203.0.113.10/32
```

Rules:

```text
90   DENY    203.0.113.10/32
100  ALLOW   0.0.0.0/0
```

Because **90 < 100**, the blocked IP is denied first.

---

# 10. NACL Association

```text
VPC
 │
 ├── NACL-A
 │     ├── Subnet-1
 │     └── Subnet-2
 │
 └── NACL-B
       └── Subnet-3
```

Remember:

- **One subnet → one NACL at a time**
- **One NACL → multiple subnets**

---

# 11. Most Important Points

```text
              NACL
                │
      ┌─────────┼─────────┐
      │         │         │
    Subnet   Stateless  Allow/Deny
      │         │         │
      │         │      Rule Number
      │         │         │
      │         │     Lowest First
      │         │
      │     Check Inbound
      │          +
      │     Check Outbound
      │
      └── Protects subnet traffic
```

### 5 Concepts Students Must Remember

1. **NACL works at subnet level.**
2. **NACL is stateless.**
3. **NACL supports ALLOW and DENY.**
4. **Lowest-number matching rule wins.**
5. **Inbound and outbound traffic are evaluated separately.**
