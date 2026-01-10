why do servers send both public key and cert if cert already contains public key along identity info all signed by CA?


Short answer: they usually don’t send both independently—what you’re seeing is a matter of protocol structure and terminology, not duplication of cryptographic meaning.

Let’s break it down cleanly.

1. A certificate already contains the public key (you’re right)

An X.509 certificate includes:

Subject identity (DNS name, org, etc.)

Subject public key

Issuer (CA)

Validity period

Extensions

CA’s signature over all of the above

So cryptographically, the certificate is the public key + identity + CA attestation.

2. Why it looks like servers send both

This confusion usually comes from one of these cases:

A. Protocol fields are named separately

Protocols often conceptually separate:

“Here is my certificate”

“Here is the public key I will use”

Even if, under the hood, the public key is extracted from the certificate.

Example: TLS APIs and logs often expose:

certificate

server public key

But the second is parsed from the first, not transmitted twice.

B. Certificate chain vs. leaf key

When a server sends its certs, it sends:

Leaf certificate (contains public key)

Intermediate certificates (no new public key for the server)

People sometimes describe this as:

“server sends cert + public key”

But the only server public key is inside the leaf certificate.

C. Different keys for different purposes (important case)

Sometimes you really do see multiple keys, but they serve different roles:

Example: TLS with ephemeral key exchange

Certificate public key
→ Authentication (proves identity)

Ephemeral key (ECDHE, etc.)
→ Key agreement (session encryption)

In TLS:

The certificate key signs something

The ephemeral key is sent separately and used to derive session keys

These are intentionally different keys.

So you might see:

Certificate with public key A

Separate public key B used for key exchange

This is not redundancy—it’s forward secrecy.

D. SSH: host key vs host certificate

In SSH:

Host key = server’s long-term public key

Host certificate = CA-signed wrapper around that host key

The server sends:

The host public key

Optionally, a certificate that certifies that key

Again:

Only one cryptographic key

Certificate just vouches for it

3. Why protocols keep them conceptually separate

Even though the cert contains the key, protocols separate the ideas because:

1️⃣ Flexibility

Some modes use raw public keys (no certs)

Some use certs

Some rotate ephemeral keys per session

2️⃣ Clean trust model

Certificate = “Why should I trust this key?”

Public key = “This is the key I’m actually using”

3️⃣ Historical reasons

Early protocols evolved incrementally

Backward compatibility matters

APIs mirror conceptual roles, not byte-level redundancy

4. When would sending both actually be redundant?

Almost never on the wire.

If you ever see:

Certificate

Exact same public key again as a separate blob

That’s usually:

Debug output

API abstraction

Human-readable logging

Not real network duplication.

5. One-sentence summary

The certificate is the public key plus identity and trust; protocols may refer to them separately, but on the wire the public key usually comes from the certificate, unless a different key is being used for a different cryptographic purpose.

If you want, I can:

Walk through a TLS handshake byte by byte

Explain SSH host certificates vs TLS certs

Compare raw public keys vs certificates in modern protocols

so in case of SSH - physically, the public host key is sent twice right?

No — in SSH the public host key is not sent twice on the wire.
It’s sent once, and the certificate (if used) is just an alternative encoding of that same key.

Let’s be very precise and protocol-level accurate.

1. What SSH actually sends during key exchange

During SSH key exchange, the server sends exactly one “host key” object in the KEX_REPLY.

That object is either:

Case A — Plain host key (no certificates)
server_host_key = public_key_blob

Case B — Host certificate
server_host_key = certificate_blob


Not both.

The protocol field is literally called server_host_key, and it holds one blob.

2. What’s inside an SSH host certificate

An SSH host certificate contains:

The same public host key

Identity info (principals, hostname patterns)

Validity period

Extensions

CA signature over all of that

So in certificate mode:

The certificate replaces the raw public key

The client extracts the public key from inside the cert

There is no second transmission of the same key.

3. Why it feels like it’s sent twice

This confusion usually comes from tooling and config, not the wire protocol:

A. known_hosts format

Example:

@cert-authority *.infra.example.com ssh-ed25519 AAAAC3...


Here you see:

Key type

Base64 blob

But this blob is the CA public key, not the host key.

The actual host public key is inside the cert the server sends.

B. Logs / API abstractions

SSH libraries often expose:

server_public_key

server_certificate

But internally:

The public key is parsed from the cert

No duplication occurred on the network

C. Signature verification step

During KEX:

Server signs the exchange hash using the private host key

Client verifies it using the public key extracted from the cert

Again:

One key

Two conceptual roles (identity + signature verification)

4. Packet-level summary (important)
Mode	What server sends
No certs	Raw host public key
With certs	Host certificate (contains the public key)
Both	❌ Impossible per SSH protocol

There is no SSH message where the server sends:

public key blob
and

certificate blob

as two separate fields.

5. The one place SSH sends “two keys” (but not the host key)

SSH does send:

Ephemeral key (for Diffie–Hellman / ECDH)

Host key (or certificate)

These are:

Cryptographically different

Used for different purposes

This is often misinterpreted as duplication.
