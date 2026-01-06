MITM in corporate network

corporate networks feel trusted, yet in practice they are one of the most common places MITM actually happens.

High-level answer

In corporate networks, MITM usually happens because:

The network is shared, centrally managed, and already designed to intercept traffic.

That’s very different from a home network.

The most common MITM mechanisms in corporate environments
1️⃣ TLS inspection proxies (the #1 reason)
What they are

Many companies deploy:

“Security gateways”

“Web proxies”

“DLP / IDS / IPS appliances”

These systems intentionally MITM traffic.

How it works
Laptop → Corporate Proxy → Internet


The proxy:

Terminates TLS

Inspects decrypted traffic

Re-encrypts it

To make this work:

Company installs a custom root CA on your machine

Browsers trust it automatically

Why this matters for SSH

SSH does not trust corporate CAs

SSH does not allow silent interception

MITM attempt → fingerprint mismatch → SSH aborts

This is why SSH often “breaks” on corporate networks — by design.

2️⃣ ARP spoofing inside the LAN
Why this works in offices

Flat VLANs

Shared Wi-Fi

Guest networks bridged to corp LAN

Misconfigured switches

An attacker machine says:

“I am the gateway.”

Other machines believe it.

Result
You → Attacker → Router → Internet


From here:

SSH MITM possible

HTTPS MITM possible (if cert trust broken)

3️⃣ Compromised internal machines

One compromised laptop:

Can ARP spoof

Can run rogue DHCP

Can poison DNS

Corporate networks assume:

“Everyone inside is trusted”

That assumption is often wrong.

4️⃣ DNS poisoning / split-horizon DNS

Corporate DNS servers:

Control internal name resolution

Often override public DNS

If DNS is compromised or misconfigured:

github.com → attacker IP


SSH catches this
HTTPS may not (if CA trust abused)

5️⃣ VPN endpoints (very important)

When you connect to a corporate VPN:

All traffic routes through VPN gateway

Gateway can inspect or redirect traffic

Gateway may terminate TLS

From your machine’s point of view:

The VPN is the network.

If VPN is compromised or malicious:

MITM is trivial

6️⃣ “Legitimate” admin access

This one is subtle but real.

Corporate admins often:

Have switch access

Have firewall access

Have routing control

They can:

Mirror traffic

Inject routes

Redirect traffic temporarily

This is not theoretical — it’s operational reality.

Why SSH is especially noisy in corporate networks

Because SSH:

Pins server identity

Has no CA override

Detects interception immediately

So you see:

REMOTE HOST IDENTIFICATION HAS CHANGED


While HTTPS:

Quietly works

Or shows generic cert warnings users click through

Why companies tolerate MITM internally

From their perspective:

Prevent data exfiltration

Enforce compliance

Monitor malware

Inspect encrypted traffic

From a security purist’s perspective:

It weakens end-to-end trust

Expands attack surface

Both views exist.

Why infra engineers care deeply

Inside corporate networks:

SSH is often the last line of defense

If host key checking is disabled:

MITM becomes invisible

Lateral movement is easy

This is how breaches spread.

One-sentence mental model

Corporate networks are trusted for convenience, not because they are cryptographically safe.

