Use Ed25519 unless you need legacy compatibility.

1. Cryptographic foundations (why they’re different)
RSA

Based on integer factorization

Security depends on large key sizes

Older (1977)

More math-heavy → slower operations

Ed25519

Based on elliptic curve cryptography (ECC)

Uses Curve25519

Designed for modern safety guarantees

Resistant to many implementation pitfalls


2. Security comparison
RSA

2048-bit = minimum acceptable today

4096-bit = safer but slow

Vulnerable to:

Bad randomness

Padding attacks

Side-channel leaks

Ed25519

Fixed, safe parameters (no tuning mistakes)

Deterministic signatures (no RNG disasters)

Very hard to misuse accidentally

 Ed25519 is more secure per line of code written.

3. Performance
Signing & verification

Ed25519: extremely fast

RSA: slower, especially with large keys

Practical impact

SSH login

Git operations

CI pipelines

Kubernetes node bootstrapping

On busy systems (CI, clusters), Ed25519 scales much better.

5. SSH-specific comparison (very relevant for you)
Ed25519 in SSH
ssh-keygen -t ed25519


Faster authentication

Smaller authorized_keys

Default on modern OpenSSH

RSA in SSH
ssh-keygen -t rsa -b 4096


Needed for:

Very old systems

Legacy appliances

Otherwise: unnecessary overhead

6. Compatibility concerns (when RSA still matters)

Use RSA only if:

You must connect to very old SSH servers

Hardware security modules don’t support Ed25519

Some legacy PKI tooling requires RSA

Otherwise → Ed25519 wins.

Best practice

SSH keys: Ed25519

Git signing: Ed25519

Cluster bootstrapping: Ed25519

Automation: Ed25519
