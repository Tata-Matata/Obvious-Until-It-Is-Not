Out of band (often written OOB) is a security term meaning:

Verifying or exchanging information via a separate, independent channel — not the one you’re trying to secure.

In other words:

In-band = over the same connection you’re establishing

Out-of-band = via a different path that an attacker would have to compromise as well

Why “out of band” matters (intuition first)

Imagine this question:

“Is the person on the phone really my bank?”

If you ask the caller to confirm their identity → useless
If you hang up and call the official number on the card → out of band

Same idea in crypto and networking.

SSH fingerprint verification (classic OOB case)
In-band (unsafe)
SSH server says: “My fingerprint is X”
You believe it


If there’s a MITM, the attacker controls that message.

Out-of-band (safe)
SSH shows fingerprint X
You compare it with:
- cloud provider console
- printed docs
- secure chat
- provider website over HTTPS


Now the attacker must compromise both channels.

Git hosting example

Providers publish SSH fingerprints:

GitHub docs

GitLab docs

You verify:

SSH output fingerprint
≟
Fingerprint on provider website


The website is secured by TLS + CA trust, which is a different trust system than SSH.

That separation is the whole point.

HTTPS example

When you see:

Your connection is not private


You might:

Check certificate fingerprint via browser tools

Compare it to one sent by the admin via Signal

Validate it against documentation

Again: out of band.

Certificate pinning as preloaded out-of-band trust

Pinning is basically:

“We did the out-of-band verification before shipping the client.”

The pin is:

Hardcoded

Or delivered via a secure update channel

At runtime, no online authority can override it.

In-band vs out-of-band (clean comparison)
Aspect	In-band	Out of band
Channel	Same connection	Separate channel
Attacker effort	Low	Much higher
Trust bootstrapping	Weak	Strong
Used for	Convenience	Security
Common out-of-band channels

Provider web console

Documentation website

Secure messaging (Signal, Matrix)

Phone call

Physical access (screen, label)

Cloud-init logs

Terraform outputs

What matters is independence, not perfection.
