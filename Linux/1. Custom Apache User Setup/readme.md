# KodeKloud Linux Journey 🐧

## Challenge 1: Creating a Custom Apache User (Beginner Perspective)

---

## 1. Starting Point: Doubt & Confusion

This was my **first-ever Linux challenge**. Before starting, I had genuine doubts:

* *Is this really meant for beginners?*
* *Am I supposed to already know Linux commands?*
* *Why does this feel like real server work instead of a tutorial?*

The challenge statement itself sounded very enterprise-level:

> In response to heightened security concerns, the xFusionCorp Industries security team has opted for custom Apache users for their web applications...

At first glance, this felt intimidating. But instead of stopping, I decided to **continue and learn by doing**.

---

## 2. First Login: Landing on `thor@jumphost`

After logging in, I always landed here:

```bash
thor@jumphost ~$
```

This immediately raised questions:

* Why am I not on the app server?
* What is a jump host?
* Who is `thor`?

### Understanding the Jump Host

I learned that:

* `jumphost` is a **bastion (gateway) server**
* It is the **only publicly accessible server**
* All internal servers (app servers, DB servers) are accessed **through it**

This mirrors real-world enterprise security architecture.

To verify my environment, I ran:

```bash
pwd
whoami
hostname
```

Which confirmed:

* User: `thor`
* Directory: `/home/thor`
* Host: `jumphost.stratos.xfusioncorp.com`

This told me clearly: **I had not started the challenge yet.**

---

## 3. Understanding the Architecture

Instead of a single server, KodeKloud uses **multiple independent servers**:

* `stapp01`
* `stapp02`
* `stapp03`

Each server is cloned from a **common base image**, which explains why some users exist across multiple servers.

Important realization:

> These are NOT sub-servers. They are separate machines created from the same template.

---

## 4. SSH Attempt #1 – Confusion Begins

The task required work on **App Server 3**.

I attempted:

```bash
ssh tony@stapp03
```

### SSH Host Authenticity Warning

I encountered:

> The authenticity of host 'stapp03' can't be established...

This was my first exposure to **SSH host key verification**.

I learned:

* This happens on **first-time connections**
* SSH is protecting against man-in-the-middle attacks
* Saying `yes` stores the server fingerprint in `~/.ssh/known_hosts`

### SSH Host Key Verification

When connecting to `stapp03` for the first time, SSH displayed
a host authenticity warning.

This occurs because the SSH client has never seen the server’s
host key before. SSH uses host keys to uniquely identify servers
and protect against man-in-the-middle attacks.

After confirming the connection, the server’s host key was
stored in `~/.ssh/known_hosts`. Future connections to the same
server will not prompt this warning unless the host key changes.
11️⃣ One-line memory rule (keep this forever)

SSH warnings are about the server’s identity — not your IP, not your user.

This is a **very sharp question**, and the confusion is *totally justified*.
The short answer feels like it *should* be “yes”… but the correct answer is **no** — and the reason is **really important**.

Let’s clear it up cleanly.

---

## Short, direct answer

👉 **No — SSH does NOT automatically recognize `stapp01/02/03`, even though they are already designed and running.**

And that’s **by design**, for security reasons.

---

## Why this feels confusing (your logic is valid)

Your thinking is:

> “These servers already exist inside the system.
> So SSH should already know them, right?”

That would make sense **if SSH were centralized**.

But SSH is **not**.

---

## The key concept 🔑 (this unlocks everything)

> **SSH trust is LOCAL, not global.**

Meaning:

* SSH does **not** trust servers just because *they exist*
* SSH only trusts servers it has **personally met before**

---

## Who is doing the SSH connection here?

This is the critical point:

You are not SSH-ing from your laptop.

You are SSH-ing from:

```
thor@jumphost
```

So the SSH client that matters is:

```
/home/thor/.ssh/
```

That client had **never connected to `stapp03` before**.

So SSH correctly said:

> “I don’t know this server yet.”

---

## Visualize the trust model 🧠

```
Your Laptop        (irrelevant)
     ❌

jumphost (thor) ─── SSH ─── stapp03
   ↑
   SSH client lives here
```

Trust is stored **per client**, not per network.

---

## Why SSH works this way (security reason)

If SSH trusted servers automatically just because:

* They’re in the same network
* They’re “pre-designed”

Then an attacker inside the network could:

* Spin up a fake server
* Pretend to be `stapp03`
* Steal credentials

SSH avoids this by saying:

> “I trust **only what I’ve verified before**.”

---

## What “designed” really means here

When KodeKloud says servers are designed:

* They exist
* They are reachable
* They are configured

It does **NOT** mean:

* Every client already trusts them
* Their host keys are pre-shared

That trust must be built **connection by connection**.

---

## What happened after you typed `yes`?

SSH did this:

1. Saved `stapp03`’s host key into:

   ```bash
   /home/thor/.ssh/known_hosts
   ```
2. From that moment on:

   * `thor@jumphost` trusts `stapp03`
   * Future connections are silent

---

## What if another user connects?

Example:

```bash
ssh banner@stapp03
```

This uses:

```bash
/home/banner/.ssh/known_hosts
```

Different file → **trust must be built again**.

---

## Enterprise-grade insight (this is big)

In real companies:

* Every admin
* Every CI server
* Every automation agent

Has its **own known_hosts** file.

That’s why:

* Host keys are distributed centrally
* Or managed via configuration management

---

## How to phrase this correctly in your GitHub doc

```md
### Why SSH Did Not Recognize Pre-Existing Servers

Although the app servers were already provisioned,
SSH trust is not automatic.

SSH stores server trust locally on the client machine.
Since the jump host had never connected to `stapp03`
before, SSH correctly prompted for host verification.

This behavior is intentional and critical for preventing
man-in-the-middle attacks.
```

---

## One-sentence rule (remember this forever)

> **Servers can exist without being trusted — trust is built per SSH client.**

---

## 5. Authentication Failure & Big Question

After accepting the host key, I was asked for a password — and it failed.

```text
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)
```

This raised a **very important doubt**:

> If `tony` is not the user for `stapp03`, why did SSH even try?

---

## 6. Critical Insight: Assigned User vs Existing User

From the server details:

| Server  | User   |
| ------- | ------ |
| stapp01 | tony   |
| stapp02 | steve  |
| stapp03 | banner |

Yet `ssh tony@stapp03` did not immediately fail.

### What I Learned

* SSH only checks **whether a user exists**, not whether it is the *correct* user
* Linux separates:

  * User existence
  * Authentication
  * Authorization

So:

> `tony` existed on `stapp03`, but was not authorized to log in.

This happens because servers are cloned from a base image.

---

## 7. Correct Access: Using the Right User

Once I used the correct credentials:

```bash
ssh banner@stapp03
```

I successfully logged in:

```bash
[banner@stapp03 ~]$
```

This confirmed:

* Correct server
* Correct user
* Ready to execute the task

---

## 8. The Core Task: Creating the User

### Challenge Requirements

* Username: `kareem`
* UID: `1076`
* Home directory: `/var/www/kareem`

Instead of using a one-liner immediately, I chose a **step-by-step approach** to truly understand Linux.

### Step-by-Step Implementation

```bash
sudo useradd kareem
sudo usermod -u 1076 kareem
sudo mkdir -p /var/www/kareem
sudo usermod -d /var/www/kareem kareem
sudo chown kareem:kareem /var/www/kareem
```

---

## 9. UID vs GID – Another Learning Moment

After verification:

```bash
id kareem
```

Output:

```text
uid=1076(kareem) gid=1002(kareem)
```

This raised another doubt:

> Why did UID change but not GID?

### Insight

* UID and GID are **independent**
* Linux does not automatically change group IDs
* The challenge did **not require GID changes**

So this state was perfectly valid.

---

## 10. Final Verification

```bash
ls -ld /var/www/kareem
```

Output:

```text
drwxr-xr-x 2 kareem kareem 4096 Feb  1 05:02 /var/www/kareem
```

### Breaking This Down

* `d` → directory
* `rwx` → full access for owner
* `r-x` → read & enter for group
* `r-x` → read & enter for others
* Owned by `kareem:kareem`

This permission structure is **ideal for Apache web directories**.

---

## 11. Final Outcome 🎉

✔ Challenge completed successfully
✔ Requirements fully met
✔ No unnecessary changes made

---

## 12. Key Learnings from My First Linux Challenge

* Jump host architecture
* SSH host verification
* Authentication vs authorization
* Base image cloning
* Linux user & group management
* UID vs GID behavior
* Directory permissions
* Importance of verification

---

## 13. Reflection

This challenge proved that:

> You do NOT need prior Linux mastery to start.
> You need curiosity, patience, and willingness to debug.

What felt intimidating at first became **clear and logical** once broken down.

This repository will continue documenting my Linux journey — confusion included — because that’s how real learning happens.

---

🚀 *On to the next challenge.*

