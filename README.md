# SwiftVerify
Lightweight authentication service that generates "disappearing" passwords.

#**Title: SwiftVerify: A Time-Sensitive OTP Engine**

**Description:**
A robust backend implementation of a One-Time Password (OTP) system designed for high-security authentication. This project demonstrates the lifecycle of a dynamic credential: from secure generation and cryptographic hashing to strictly enforced expiration and single-use invalidation logic.

**Key Features:**
**Temporal Expiry:** Passwords automatically invalidate after a 120-second window.
**Atomic Invalidation:** "Burn-on-use" logic to prevent replay attacks.
**Secure Storage:** Implements one-way hashing (Argon2/Bcrypt) to ensure zero plain-text exposure.
**Rate Limiting:** Built-in protection against brute-force guessing.
