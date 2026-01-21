First: what does “changed” mean in Ansible?

Ansible tracks state changes, not just command execution.

For every task, Ansible asks:

“Did this task modify the system?”

The answer affects:

play recap (ok vs changed)

handlers (they run only when something changed)

idempotency reporting

CI/CD logic

What Ansible does by default

For modules (like sysctl, apt):

Ansible knows whether a change happened

For command/shell:

Ansible has no idea

So it assumes:

“Command ran → system probably changed”

That assumption is often wrong.

Now your task specifically
The check task
- name: Check forward rule public → private
  command: iptables -C FORWARD -i eth0 -o enp7s0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
  register: fwd_in_check
  failed_when: false
  changed_when: false

What this command does

iptables -C ...:

Only checks

Never modifies anything

Either:

rule exists → exit 0

rule missing → exit 1

No system change is possible. Ever.

What Ansible would think without changed_when: false

Without it, Ansible sees:

Command executed


So it reports:

changed


Even though nothing changed.

That leads to:

misleading output

handlers firing incorrectly

“changed” every run

broken idempotency

What changed_when: false actually means

It literally tells Ansible:

“No matter what happens here, this task did NOT change the system.”

So Ansible reports:

ok


instead of:

changed

Concrete before/after example
❌ Without changed_when: false
TASK [Check forward rule public → private]
changed


Every run looks like it modified the system — lies.

✅ With changed_when: false
TASK [Check forward rule public → private]
ok


Correct and honest.

Why this matters more than it seems
1️⃣ Accurate reporting

When you later see:

changed=3


You can trust that something actually changed.

2️⃣ Handlers depend on it

Handlers run only if a task reports changed.

If you don’t use changed_when: false:

your “save iptables rules” handler

or “restart service” handler
will run every time

Bad.

3️⃣ Idempotency is not just behavior — it’s reporting

A playbook that:

does nothing

but still says “changed”

is not idempotent in Ansible terms.

Mental model (this makes it click)

Think of it like this:

Task type	Should it ever change state?	changed_when
Check-only command	❌ No	false
Read-only query	❌ No	false
Enforcement command	✅ Sometimes	default
State-setting module	Ansible decides	not needed

Your iptables -C task is pure observation.

So we tell Ansible that explicitly.

Why we don’t do changed_when: fwd_in_check.rc != 0

Because:

the check task still didn’t change anything

the apply task does the change

responsibilities stay clean

Summary in one sentence

changed_when: false tells Ansible: “this task only checks state — it never modifies the system.”
