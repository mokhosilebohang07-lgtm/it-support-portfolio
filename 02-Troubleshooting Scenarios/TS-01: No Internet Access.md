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
### Step 2: Test Local Network Stack

```cmd
ping 127.168.1.0

