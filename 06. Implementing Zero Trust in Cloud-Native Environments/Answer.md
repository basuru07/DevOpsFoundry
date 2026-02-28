# SWIFT Payment Security: Bangladesh Bank Heist & Zero Trust Analysis

---

## 1. Incident Analysis

### What Happened in the Bangladesh Bank Heist (2016)

In February 2016, attackers compromised the internal systems of Bangladesh Bank, the country's central bank. They gained access to the bank's SWIFT transaction environment and used stolen credentials to send fraudulent money transfer requests through the SWIFT network.

The attackers attempted to steal nearly $1 billion by issuing fake transfer orders to the Federal Reserve Bank of New York. While many transactions were blocked due to errors, **$81 million** was successfully transferred to accounts in the Philippines and laundered through casinos.

The attack succeeded mainly because once the attackers entered the internal network, systems trusted each other implicitly, and there were no strong internal security controls to stop or detect abnormal behavior.

### Major Security Weaknesses

**1. Implicit Trust Inside the Network**
Internal systems trusted authenticated users and devices without continuous verification.

**2. Weak Monitoring and Logging**
Critical systems (including the SWIFT interface) had poor or no real-time monitoring, and some printers/logging systems were disabled, delaying detection.

**3. Credential-Based Access Without Strong Controls**
Stolen credentials were enough to authorize high-value financial transactions, with no multi-factor checks or behavioral validation.

---

## 2. Zero Trust-Based Recommendations

> Zero Trust follows the principle: **"Never trust, always verify."**

### Recommendation 1: Strong Identity Verification with Continuous Authentication

**Zero Trust Principle:** Verify explicitly (every user, every request)

Even if attackers stole credentials, they should not have been able to access or use the SWIFT system without additional verification.

**Controls & Tools:**
- Multi-Factor Authentication (MFA) for all SWIFT transactions
- Device-based authentication (certificates, device fingerprinting)
- Privileged Access Management (PAM) for high-risk accounts

**Impact:** Stolen usernames/passwords alone would not have been sufficient to authorize fraudulent transfers.

---

### Recommendation 2: Network Micro-Segmentation

**Zero Trust Principle:** Assume breach and limit blast radius

Once attackers entered the network, they could freely access critical systems. Micro-segmentation would isolate sensitive systems like SWIFT from general IT infrastructure.

**Controls & Tools:**
- Micro-segmentation using firewalls or software-defined networking
- Separate secure zone for SWIFT systems
- Strict east-west traffic controls

**Impact:** Even if attackers compromised one system, they could not move laterally to the SWIFT environment.

---

### Recommendation 3: Continuous Monitoring and Behavioral Analytics

**Zero Trust Principle:** Continuous monitoring and validation

The fraudulent transactions were unusual in size, timing, and destination. Real-time monitoring could have flagged them immediately.

**Controls & Tools:**
- Security Information and Event Management (SIEM)
- User and Entity Behavior Analytics (UEBA)
- Real-time alerts for abnormal transaction patterns

**Impact:** Suspicious transactions could have been detected and stopped before money was transferred.

---

## 3. High-Level Zero Trust Architecture Diagram

```
            ┌──────────────────────┐
            │   User / Operator     │
            └─────────┬────────────┘
                      │
              (MFA + Device Check)
                      │
            ┌─────────▼────────────┐
            │ Identity & Access     │
            │ Management (IAM)      │
            └─────────┬────────────┘
                      │
        ┌─────────────▼─────────────┐
        │ Policy Enforcement Point  │
        │ (Zero Trust Gateway)      │
        └───────┬─────────┬────────┘
                │         │
      ┌─────────▼───┐  ┌──▼──────────┐
      │ SWIFT Zone  │  │ Core Banking │
      │ (Isolated)  │  │ Systems     │
      └─────────────┘  └─────────────┘
                │
        ┌───────▼────────┐
        │ Monitoring &   │
        │ SIEM / UEBA    │
        └────────────────┘
```

---

## Conclusion

The Bangladesh Bank Heist clearly demonstrates the dangers of implicit trust in internal networks. If Zero Trust Architecture had been applied — through strong identity verification, micro-segmentation, and continuous monitoring — the attack could have been prevented or significantly mitigated.

This case strongly supports why modern financial institutions must adopt Zero Trust to protect high-value systems and transactions.
