# Windows Server Administration 

Practical Windows Server administration labs covering **DHCP, DHCP scopes, DHCP failover, DNS, forward lookup zones, and reverse lookup zones**.

---

## 📚 Labs Included

| #  | Lab                                              | Description                                |
| -- | ------------------------------------------------ | ------------------------------------------ |
| 01 | [DHCP Installation](./01-dhcp-installation/)     | Install and configure the DHCP Server role |
| 02 | [DHCP Scope](./02-dhcp-scope/)                   | Create and configure a DHCP scope          |
| 03 | [DHCP Failover](./03-dhcp-failover/)             | Configure DHCP high availability           |
| 04 | [DNS Installation](./04-dns-installation/)       | Install the DNS Server role                |
| 05 | [Forward Lookup Zone](./05-forward-lookup-zone/) | Configure name-to-IP resolution            |
| 06 | [Reverse Lookup Zone](./06-reverse-lookup-zone/) | Configure IP-to-name resolution            |

---

# 🖥️ Lab Environment

The lab uses multiple virtual machines to simulate a small Windows Server network.

### Virtual Machines

| VM       | Operating System | IP Address     | Purpose           |
| -------- | ---------------- | -------------- | ----------------- |
| SERVER01 | Windows Server   | `192.168.1.10` | DHCP + DNS        |
| SERVER02 | Windows Server   | `192.168.1.11` | DHCP Failover     |
| CLIENT01 | Windows 10/11    | DHCP           | DHCP/DNS testing  |
| CLIENT02 | Windows 10/11    | DHCP           | Additional client |

### Network

```text
                    Virtual Network
                  192.168.1.0/24
                         |
          +--------------+--------------+
          |                             |
      SERVER01                      SERVER02
    192.168.1.10                  192.168.1.11
      DHCP + DNS                  DHCP Failover
          |                             |
          +--------------+--------------+
                         |
                      CLIENT01
                        DHCP
                         |
                  192.168.1.100+
```

---

# 🎯 Learning Objectives

By completing these labs, you should be able to:

* Install Windows Server DHCP.
* Configure a DHCP scope.
* Configure DHCP options.
* Test DHCP from a Windows client.
* Configure DHCP failover.
* Understand Load Balance and Hot Standby modes.
* Install Windows Server DNS.
* Create a forward lookup zone.
* Create a reverse lookup zone.
* Create A and PTR records.
* Test DNS using `nslookup`.
* Troubleshoot basic DHCP and DNS connectivity.

---

# 🔧 Prerequisites

Before starting, make sure you have:

* [ ] Windows Server virtual machine
* [ ] Windows client virtual machine
* [ ] A virtual network/switch connecting the VMs
* [ ] Administrator access
* [ ] Static IP configured on the servers
* [ ] Two Windows Server VMs for the DHCP failover lab

### Suggested IP Configuration

**SERVER01**

```text
IP Address:       192.168.1.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
DNS Server:       192.168.1.10
```

**SERVER02**

```text
IP Address:       192.168.1.11
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
DNS Server:       192.168.1.10
```

---

# 01 – DHCP Installation

## Objective

Install the DHCP Server role on Windows Server using Server Manager.

## Why DHCP?

DHCP automatically provides network configuration to clients.

Instead of manually configuring:

```text
IP Address
Subnet Mask
Default Gateway
DNS Server
```

DHCP provides this information automatically.

---

## Steps

### 1. Configure a static IP

On `SERVER01`:

```text
IP Address:       192.168.1.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
Preferred DNS:    192.168.1.10
```

### Why?

Infrastructure servers should have predictable IP addresses.

---

### 2. Open Server Manager

Go to:

```text
Server Manager
    ↓
Manage
    ↓
Add Roles and Features
```

---

### 3. Select installation type

Choose:

**Role-based or feature-based installation**

Click **Next**.

---

### 4. Select SERVER01

Select:

```text
SERVER01
```

Click **Next**.

---

### 5. Install DHCP Server

Under **Server Roles**:

* Select **DHCP Server**
* Select **Add Features** when prompted
* Click **Next**
* Click **Install**

---

### 6. Complete DHCP configuration

After installation:

```text
Server Manager
    ↓
Notifications
    ↓
Complete DHCP configuration
```

Complete the configuration wizard.

---

## Verification

Open:

```text
Server Manager
    ↓
Tools
    ↓
DHCP
```

You should see:

```text
DHCP
└── SERVER01
    ├── IPv4
    └── IPv6
```

---

# 02 – DHCP Scope

## Objective

Create a DHCP scope that distributes IP addresses to clients.

## Example Scope

```text
Network:       192.168.1.0/24
Start IP:      192.168.1.100
End IP:        192.168.1.200
Subnet Mask:   255.255.255.0
```

---

## Steps

### 1. Open DHCP Manager

```text
Server Manager
    ↓
Tools
    ↓
DHCP
```

---

### 2. Create a new scope

Expand:

```text
SERVER01
└── IPv4
```

Right-click **IPv4**.

Select:

**New Scope**

---

### 3. Name the scope

Example:

```text
LAN Scope
```

---

### 4. Configure the IP range

Enter:

```text
Start IP: 192.168.1.100
End IP:   192.168.1.200
```

Subnet mask:

```text
255.255.255.0
```

---

### 5. Configure exclusions

Example:

```text
192.168.1.100 - 192.168.1.110
```

This leaves:

```text
192.168.1.111 - 192.168.1.200
```

available for DHCP clients.

### Why?

Static devices such as servers, printers, and network equipment should not accidentally receive addresses from DHCP.

---

### 6. Configure lease duration

For the lab:

```text
8 days
```

---

### 7. Configure the default gateway

Enter:

```text
192.168.1.1
```

### Why?

This tells clients where to send traffic destined for another network.

---

### 8. Configure DNS

Enter:

```text
192.168.1.10
```

### Why?

Clients use the DNS server to resolve hostnames into IP addresses.

---

### 9. Activate the scope

Select:

**Yes, I want to activate this scope now**

Click **Finish**.

---

## 🧪 Test DHCP

On `CLIENT01`:

```cmd
ipconfig /release
ipconfig /renew
```

Then:

```cmd
ipconfig /all
```

Expected:

```text
IPv4 Address:     192.168.1.xxx
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
DNS Server:       192.168.1.10
```

---

# 03 – DHCP Failover

## Objective

Configure DHCP high availability using two DHCP servers.

### Topology

```text
                 Network
                    |
          +---------+---------+
          |                   |
      SERVER01            SERVER02
    192.168.1.10        192.168.1.11
       DHCP                DHCP
          \                 /
           \--- Failover --/
```

## Why?

If `SERVER01` fails, `SERVER02` can continue providing DHCP service.

---

## Steps

### 1. Configure SERVER02

Set:

```text
IP Address:       192.168.1.11
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
DNS Server:       192.168.1.10
```

---

### 2. Install DHCP on SERVER02

Repeat the DHCP installation process:

```text
Server Manager
    ↓
Manage
    ↓
Add Roles and Features
    ↓
DHCP Server
    ↓
Install
```

---

### 3. Open DHCP on SERVER01

```text
Server Manager
    ↓
Tools
    ↓
DHCP
    ↓
SERVER01
    ↓
IPv4
```

---

### 4. Configure failover

Right-click **IPv4**.

Select:

**Configure Failover**

Select the DHCP scope.

Click **Next**.

---

### 5. Select SERVER02

Enter:

```text
SERVER02
```

or:

```text
192.168.1.11
```

---

### 6. Select failover mode

#### Load Balance

Both servers actively provide DHCP services.

```text
             Clients
             /     \
            /       \
       SERVER01   SERVER02
         DHCP       DHCP
```

#### Hot Standby

One server is active while the other acts as a backup.

```text
CLIENTS
   |
SERVER01
 ACTIVE
   |
 Failure
   |
SERVER02
 STANDBY → ACTIVE
```

---

### 7. Configure the relationship

Example:

```text
Relationship:
DHCP-Failover

Mode:
Load Balance
```

Complete the wizard.

---

## 🧪 Test Failover

On `CLIENT01`:

```cmd
ipconfig /release
ipconfig /renew
```

Confirm that the client receives an IP address.

Then stop the DHCP service on `SERVER01`.

Try:

```cmd
ipconfig /renew
```

The client should still receive DHCP configuration from `SERVER02`.

---

# 04 – DNS Installation

## Objective

Install the DNS Server role.

## Why DNS?

DNS converts hostnames into IP addresses.

```text
server01.lebohang.com
        ↓
192.168.1.10
```

---

## Steps

### 1. Open Server Manager

```text
Server Manager
    ↓
Manage
    ↓
Add Roles and Features
```

---

### 2. Select installation type

Choose:

**Role-based or feature-based installation**

---

### 3. Select SERVER01

Select:

```text
SERVER01
```

---

### 4. Select DNS Server

Under **Server Roles**:

* Select **DNS Server**
* Select **Add Features** if prompted
* Click **Next**
* Click **Install**

---

### 5. Open DNS Manager

Go to:

```text
Server Manager
    ↓
Tools
    ↓
DNS
```

You should see:

```text
SERVER01
├── Forward Lookup Zones
└── Reverse Lookup Zones
```

---

# 05 – Forward Lookup Zone

## Objective

Create a DNS zone that resolves:

```text
NAME → IP
```

Example:

```text
server01.lebohang.com
        ↓
192.168.1.10
```

---

## Steps

### 1. Open DNS Manager

```text
Server Manager
    ↓
Tools
    ↓
DNS
```

---

### 2. Create the zone

Right-click:

**Forward Lookup Zones**

Select:

**New Zone**

---

### 3. Select Primary Zone

Choose:

**Primary zone**

Click **Next**.

---

### 4. Enter the zone name

Example:

```text
lab.local
```

---

### 5. Complete the wizard

Accept the appropriate default settings.

Click:

**Finish**

---

## Create an A Record

Right-click:

```text
lebohang.com
```

Select:

**New Host (A or AAAA)**

Enter:

```text
Name:
server01

IP address:
192.168.1.10
```

Click **Add Host**.

You now have:

```text
server01.lebohang.com
        ↓
192.168.1.10
```

---

## 🧪 Test Forward DNS

On `CLIENT01`:

```cmd
nslookup server01.lebohang.com
```

Expected result:

```text
Name:    server01.lebohang.com
Address: 192.168.1.10
```

You can also test:

```cmd
ping server01.lab.local
```

---

# 06 – Reverse Lookup Zone

## Objective

Create a DNS zone that resolves:

```text
IP → NAME
```

Example:

```text
192.168.1.10
      ↓
server01.lebohang.com
```

---

## Steps

### 1. Open DNS Manager

```text
Server Manager
    ↓
Tools
    ↓
DNS
```

---

### 2. Create a reverse zone

Right-click:

**Reverse Lookup Zones**

Select:

**New Zone**

---

### 3. Select Primary Zone

Choose:

**Primary zone**

---

### 4. Select IPv4

Choose:

**IPv4 Reverse Lookup Zone**

---

### 5. Enter Network ID

For:

```text
192.168.1.0/24
```

Enter:

```text
192.168.1
```

Windows will create:

```text
1.168.192.in-addr.arpa
```

---

### Why is it backwards?

Reverse DNS uses the `in-addr.arpa` namespace.

The network:

```text
192.168.1.0
```

becomes:

```text
1.168.192.in-addr.arpa
```

---

## Create a PTR Record

Right-click the reverse zone.

Select:

**New Pointer (PTR)**

Enter:

```text
Host IP:
10

Host name:
server01.lebohang.com
```

Click **OK**.

You now have:

```text
192.168.1.10
      ↓
server01.lebohang.com
```

---

## 🧪 Test Reverse DNS

Run:

```cmd
nslookup 192.168.1.10
```

Expected:

```text
Name:
server01.lebohang.com

Address:
192.168.1.10
```

---

# 🧪 Troubleshooting Commands

## Check IP configuration

```cmd
ipconfig /all
```

## Release DHCP address

```cmd
ipconfig /release
```

## Request DHCP address

```cmd
ipconfig /renew
```

## Test connectivity

```cmd
ping 192.168.1.10
```

## Test hostname resolution

```cmd
nslookup server01.lebohang.com
```

## Test reverse DNS

```cmd
nslookup 192.168.1.10
```

## Clear DNS cache

```cmd
ipconfig /flushdns
```

## View DNS cache

```cmd
ipconfig /displaydns
```

---

# 📸 Evidence / Screenshots

For each lab, screenshots should show the important configuration and verification steps.

### DHCP Installation

* [ ] DHCP role installed
* [ ] DHCP Manager showing SERVER01
* [ ] IPv4 section visible

### DHCP Scope

* [ ] Scope configuration
* [ ] Address pool
* [ ] DHCP options
* [ ] `ipconfig /all` on CLIENT01

### DHCP Failover

* [ ] SERVER01 DHCP configuration
* [ ] SERVER02 DHCP configuration
* [ ] Failover relationship
* [ ] Replicated scope
* [ ] Successful client renewal after SERVER01 failure

### DNS Installation

* [ ] DNS role installed
* [ ] DNS Manager

### Forward Lookup Zone

* [ ] Forward Lookup Zone
* [ ] A record
* [ ] Successful `nslookup`

### Reverse Lookup Zone

* [ ] Reverse Lookup Zone
* [ ] PTR record
* [ ] Successful reverse `nslookup`

#

|   |
| - |
