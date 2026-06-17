# TS-01: No Internet Access

## Scenario
A user reported that they could not access any websites despite being connected to the network.

---

## Problem
The user was unable to browse the internet.

---

## Symptoms
- Browser displayed connection errors.
- Websites failed to load.
- Network icon showed connected status.

---

## Investigation

### Step 1: Check IP Configuration

Command used:

```cmd
ipconfig
```
<img width="1363" height="176" alt="image" src="https://github.com/user-attachments/assets/9d4e9ae4-089f-4833-87ef-506cba012467" />

### Step 2: Test Local Network Stack

Command used:

```cmd
ping 127.0.0.1
```
<img width="1365" height="364" alt="image" src="https://github.com/user-attachments/assets/87bf7d7d-4501-463f-8e37-f1739271ac0b" />

### Step 3: Test Gateway Connectivity

Command used:

```cmd
ping 172.20.10.1
```
<img width="1352" height="279" alt="image" src="https://github.com/user-attachments/assets/e8490caa-2180-4bd2-9696-09feb3dfe71d" />

### Step 4: Test External Connectivity

Command Used:
```cmd
ping 8.8.8.8
```
<img width="1365" height="285" alt="image" src="https://github.com/user-attachments/assets/72aad38f-27e7-4c71-922e-58840adad568" />

### Step 4: Test DNS Resolution

Command Used:

```cmd
ping google.com
```

<img width="1357" height="72" alt="image" src="https://github.com/user-attachments/assets/5184cbfc-8f75-4067-99d5-99c31643f56c" />


## Tools Used:

- Command Prompt
- Network Adapter Settings
- DNS Configuration

## Root Cause

DNS was not resolving domain names correctly.

## Solution 

Commands Used:

```cmd
ipconfig /flushdns
ipconfig /renew
```
<img width="1365" height="115" alt="image" src="https://github.com/user-attachments/assets/9d207edf-dc00-482c-a28f-0576e0e4e371" />


## Outcome:

Internet access was restored and websites loaded successfully.

<img width="1365" height="242" alt="image" src="https://github.com/user-attachments/assets/e30d22e7-4a26-4f82-b765-a49ad5e504da" />

