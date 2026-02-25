# Implementing Zero Trust in Cloud-Native Environments

---

## 01. Traditional Security (Old Model)

### What is it?

Old security model protects the outside boundary of a company network using a firewall.

- If you are inside the network → You are trusted.
- If you are outside → You are blocked.

### Real World Example

Think about a school with a security gate.
If you enter the gate, no one checks you again.
Inside the school, you can walk anywhere.

### Problem

- If a bad person enters once, they can go everywhere.
- If firewall fails → whole system is open.

---

## 02. Why Zero Trust?

**Today:**

- People work from home
- Apps are in cloud
- Hackers steal passwords easily

So trusting people just because they are "inside network" is dangerous.

### What is Zero Trust?

**Main Idea:**

> "Never Trust, Always Verify."

Even if you are inside the network, you must prove who you are.

---

## 03. Zero Trust Architecture Components

### 1. Identity & Access Management

- SSO (Single Sign-On)
- MFA (Multi-Factor Authentication)
- RBAC (Role-Based Access Control)
- Short-lived credentials
- Certificate-based SSH

Ensures only verified users/services can access.

### 2. Device Security

Check device health before access. Ensure:

- Updated OS
- Antivirus enabled
- Encryption enabled

### 3. Network Segmentation

- Divide network into small secure zones.
- Use **Micro-segmentation**.
- Allow only required communication.
- Deny everything else.

**Example:**

- Service A can talk to Service B
- But **cannot** talk to Service C.

### 4. Application Security

- Use **mTLS** (mutual TLS)
- Encrypt all service-to-service communication.
- Use **Service Mesh** for secure communication.

### 5. Data Protection

Encrypt data:
- At rest
- In transit

Classify data (public, confidential, restricted) and use **DLP** (Data Loss Prevention).

### 6. Continuous Monitoring

- Collect logs
- Monitor CPU, memory
- Track network requests
- Detect unusual behavior
- Use **SIEM** tools

> Zero Trust requires full visibility.

---

## Example: Microservices System

If system has:

**Domain Foo:** S1, S2  
**Domain Bar:** S1, S2, S3

**Rules:**

| From | To | Allowed? |
|------|----|----------|
| Foo/S1 | Foo/S2 | ✅ |
| Bar/S2 | Bar/S3 | ✅ |
| Foo/S2 | Bar/S1 | ✅ |
| Everything else | — | ❌ DENIED |

All calls are: **Encrypted** · **Authenticated** · **Logged**

This is Zero Trust in microservices.

---

## 🏦 Bangladesh Bank Heist Example (Real Case)

**In 2016, attackers:**

- Stole credentials
- Sent fake SWIFT transfer requests
- **$81 million stolen**

**Security Weaknesses:**

- Too much internal trust
- Weak monitoring
- No proper segmentation

---

## Key Takeaways

- Perimeter security alone is not enough
- Zero Trust is not just a tool — it is a **design approach**
- Everything must be verified
- Use **least privilege**
- Encrypt everything
- Monitor continuously
- **Assume breach**
