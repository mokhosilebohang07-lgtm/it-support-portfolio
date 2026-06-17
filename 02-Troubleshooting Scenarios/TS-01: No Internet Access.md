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

```cmd
ping 127.0.0.1
```
<img width="1365" height="364" alt="image" src="https://github.com/user-attachments/assets/87bf7d7d-4501-463f-8e37-f1739271ac0b" />


