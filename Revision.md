### AWS VPC — Traffic Flow

```text
Inbound Internet Traffic

Internet
   ↓
Internet Gateway (IGW)
   ↓
Route decision / Route Table
   ↓
NACL (Subnet Boundary)
   ↓
Security Group (ENI)
   ↓
EC2
```

For outbound traffic:

```text
EC2
   ↓
Security Group
   ↓
NACL (Subnet Boundary)
   ↓
Route decision / Route Table
   ↓
Internet Gateway
   ↓
Internet
```

## Core Concepts

| Concept | Exact Purpose |
|---|---|
| **VPC** | Logically isolated network in AWS |
| **CIDR** | Defines the IP address range |
| **Subnet** | Range of IP addresses inside a VPC |
| **Public Subnet** | Subnet whose route table has a route to an IGW |
| **Private Subnet** | Subnet without a direct route to an IGW |
| **Route Table** | Determines where network traffic is routed |
| **IGW** | Provides internet connectivity for the VPC |
| **NAT Gateway** | Allows private-subnet resources to initiate outbound internet connections |
| **Security Group** | Stateful firewall associated with ENIs/resources |
| **NACL** | Stateless firewall applied at the subnet boundary |

## Correct Architecture

```text
                         INTERNET
                            ↕
                   ┌─────────────────┐
                   │ Internet Gateway│
                   │      (IGW)      │
                   └────────┬────────┘
                            │
          ┌────────────────────────────────────┐
          │          VPC 10.0.0.0/16           │
          │                                    │
          │   PUBLIC SUBNET 10.0.1.0/24        │
          │   ┌────────────────────────────┐   │
          │   │ Route Table                │   │
          │   │ 0.0.0.0/0 → IGW            │   │
          │   │                            │   │
          │   │ NACL                       │   │
          │   │                            │   │
          │   │    ┌──────────────┐        │   │
          │   │    │ Security     │        │   │
          │   │    │ Group        │        │   │
          │   │    │     ↓        │        │   │
          │   │    │    EC2       │        │   │
          │   │    └──────────────┘        │   │
          │   │                            │   │
          │   │    ┌──────────────┐        │   │
          │   │    │ NAT Gateway  │        │   │
          │   │    └──────▲───────┘        │   │
          │   └───────────│────────────────┘   │
          │               │                    │
          │               │ Internet-bound     │
          │               │ traffic            │
          │               │                    │
          │   PRIVATE SUBNET 10.0.2.0/24       │
          │   ┌───────────│────────────────┐   │
          │   │ Route Table                │   │
          │   │ 0.0.0.0/0 → NAT Gateway  ──┘   │
          │   │                            │   │
          │   │ NACL                       │   │
          │   │                            │   │
          │   │    ┌──────────────┐        │   │
          │   │    │ Security     │        │   │
          │   │    │ Group        │        │   │
          │   │    │     ↑        │        │   │
          │   │    │    EC2       │        │   │
          │   │    └──────────────┘        │   │
          │   └────────────────────────────┘   │
          │                                    │
          └────────────────────────────────────┘
```

### Public EC2 Internet Flow

```text
INBOUND
Internet
   ↓
IGW
   ↓
Route decision
   ↓
NACL
   ↓
Security Group
   ↓
EC2


OUTBOUND
EC2
   ↓
Security Group
   ↓
NACL
   ↓
Route decision
   ↓
IGW
   ↓
Internet
```

### Private EC2 Internet Flow

A private EC2 normally **initiates** the connection:

```text
Private EC2
     ↓
Security Group
     ↓
NACL
     ↓
Private Route Table
0.0.0.0/0 → NAT Gateway
     ↓
NAT Gateway
(Public Subnet)
     ↓
Public Subnet Route Table
0.0.0.0/0 → IGW
     ↓
Internet Gateway
     ↓
Internet
```

Return traffic follows the reverse path:

```text
Internet
   ↓
IGW
   ↓
NAT Gateway
   ↓
Private Subnet
   ↓
NACL
   ↓
Security Group
   ↓
Private EC2
```

## Security Group vs NACL

| Security Group | NACL |
|---|---|
| ENI/resource level | Subnet level |
| **Stateful** | **Stateless** |
| Allow rules only | Allow + Deny rules |
| No rule numbers | Rules evaluated by rule number |
| Return traffic automatically allowed | Return traffic must be explicitly allowed |
| Applies to associated ENIs | Applies to all traffic crossing subnet boundary |

### Key Point to Remember

```text
Security Group = Instance/ENI Protection
NACL           = Subnet Protection
Route Table    = Where traffic goes
IGW            = VPC ↔ Internet
NAT Gateway    = Private Subnet → Internet
```
