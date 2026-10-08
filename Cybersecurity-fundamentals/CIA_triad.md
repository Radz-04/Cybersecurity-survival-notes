# CIA Triad
## What is the CIA triad ?
The CIA Triad is a foundational model in information security that stands for Confidentiality, Integrity, and Availability.

## Confidentiality

Ensuring that sensitive information is accessible only to authorized users and protected against unauthorized disclosure.

* **Core Goal:** Protect privacy, proprietary data, and credentials from unauthorized exposure.
* **Security Measures:** Encryption, access control mechanisms, multi-factor authentication (MFA).
* **Breaches:** 
  - Unencrypted network traffic intercepted on public/shared networks.
  - Exposed private keys, API secrets, or configuration files.
  - Unauthorized access to restricted databases or internal documents.


## Integrity

Guaranteeing that data remains accurate, complete, and authentic, preventing unauthorized modification or tampering.

* **Core Goal:** Maintain data accuracy, authenticity, and consistency across its lifecycle.
* **Security Measures:** Cryptographic hashing, digital signatures, strict input validation, authorization checks.
* **Breaches:**
  - Client-side parameter or session token tampering.
  - Unauthorized modification of system files, audit logs, or database entries.
  - Malicious code injection altering application behavior or data output.


## Availability

Ensuring that systems, networks, and data remain operational and accessible to authorized users when needed.

* **Core Goal:** Prevent operational downtime, service disruption, and system lockouts.
* **Security Measures:** Infrastructure redundancy, load balancing, backups, rate limiting, traffic management.
* **Breaches:**
  - Denial of Service (DoS/DDoS) attacks crashing web applications or network infrastructure.
  - System downtime caused by hardware failure, power outages, or unhandled software crashes.
  - Storage or filesystem lockout caused by ransomware execution
