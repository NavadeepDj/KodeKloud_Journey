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

