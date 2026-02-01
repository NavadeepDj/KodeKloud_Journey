# Azure SSH Key Creation – KodeKloud 100 Days Cloud Challenge (Day 1)

> **Author:** Navadeep Marella
> **Platform:** KodeKloud – 100 Days Cloud Challenge
> **Cloud Provider:** Microsoft Azure
> **Focus Area:** DevOps Foundations – Secure Access using SSH Keys

---

## 1. Context & Problem Statement

The Nautilus DevOps team is planning a **gradual migration** of infrastructure to Azure Cloud. Instead of performing a large, risky migration in one go, they are breaking the migration into **small, controlled tasks**. This approach helps in:

* Reducing operational risk
* Improving visibility and control
* Ensuring secure and scalable infrastructure setup

### **Task Assigned**

Create an **SSH key pair** in Azure with the following constraints:

* **Key Name:** `devops-kp`
* **Key Type:** `RSA`
* **Environment:** Azure (via Azure CLI)

Azure credentials were provided via the `showcreds` command on the `azure-client` host.

---

## 2. Initial Questions & Doubts (Beginner Mindset)

Before executing the task, several foundational questions naturally arose:

* *Is this task too difficult for a beginner?*
* *Why is SSH considered a core DevOps skill?*
* *Do I need to learn DevOps theory first before doing these tasks?*
* *Why not use username/password like normal logins?*

### Key Clarification

This task is intentionally **beginner-friendly** and designed to introduce **secure infrastructure access**, which is a **non-negotiable DevOps skill**.

> DevOps is not something you learn first and apply later — you learn DevOps **by doing** tasks like this.

---

## 3. Why SSH Keys Are a Core DevOps Skill

Traditional access:

```
Username + Password
```

Modern DevOps access:

```
SSH Key Pair (Public + Private)
```

### Why SSH Keys?

* Passwords can be brute-forced or leaked
* Automation tools (CI/CD, Ansible, Terraform) cannot type passwords
* SSH keys are cryptographically secure
* Password-based SSH is often disabled in production

SSH is the **foundation** for:

* VM access
* Server configuration
* CI/CD deployments
* Infrastructure automation

---

## 4. Understanding SSH Key Pairs

An SSH key pair consists of:

* **Public Key** → Stored on the server / cloud provider
* **Private Key** → Stored securely on the user’s machine

Authentication happens when:

* The server challenges the client
* The client proves ownership of the private key
* No password is ever transmitted

> We don’t “log in” to servers anymore — we **authenticate using keys**.

---

## 5. Azure Login & Environment Setup

### Viewing Credentials

```bash
showcreds
```

This displayed:

* Azure portal URL
* Azure username
* Azure password
* Application Client ID
* Session expiry time

### Azure CLI Login

```bash
az login
```

Login used **device authentication**, followed by selecting the correct subscription:

* **Subscription:** Azure Free Labs
* **Tenant:** azurefreekmlprod

---

## 6. Identifying the Resource Group

Azure resources must belong to a **Resource Group**, which also determines the **default region**.

```bash
az group list -o table
```

Output:

```
Name                          Location    Status
kml_rg_main-9776c0c9b9344238  westus      Succeeded
```

This resource group was used for all further steps.

---

## 7. Creating the SSH Key Pair (Main Task)

### Command Used

```bash
az sshkey create \
  --name devops-kp \
  --resource-group kml_rg_main-9776c0c9b9344238
```

### Output Explanation

```
No public key is provided. A key pair is being generated for you.
```

This message caused confusion initially.

### Important Clarification

Azure supports **two workflows**:

1. You provide an existing public key → Azure stores it
2. You don’t provide a key → **Azure generates an RSA key pair for you**

This message is **informational**, not an error.

---

## 8. Where the Keys Were Stored

Azure automatically generated and saved:

* **Private Key:** `/root/.ssh/1769953461_0779624`
* **Public Key:** `/root/.ssh/1769953461_0779624.pub`

### What does “stored on my local machine” actually mean?

In this lab environment, **“local machine” refers to the Azure client VM (the Linux shell you are currently logged into)** — *not* your personal laptop or physical computer.

* The path `/root/.ssh/` is a directory on the **local filesystem of the current Linux environment**
* This storage resides on the **VM’s virtual disk (SSD-backed storage provided by Azure)**
* The private key is stored as a **regular file on disk**, just like any other file in Linux

So in simple terms:

> **Local storage = the disk of the machine where you ran the command**

In this case:

* You ran the command inside a **temporary Azure lab VM**
* The private key lives only inside that VM
* When the lab session expires, this VM (and the key) will be destroyed

### Why this is important (real-world perspective)

In real-world DevOps setups:

* If you generate a key on your **laptop**, it is stored on your laptop’s SSD
* If you generate a key on a **jump server / bastion host**, it is stored there
* The **private key never leaves the machine where it is generated** unless you explicitly copy it

Azure:

* **Never stores your private key**
* Stores **only the public key** in the cloud

This separation is a **core security principle** of SSH-based authentication.

> If Azure had your private key, SSH security would be broken.

Azure itself stores **only the public key**.

---

## 9. Verifying the SSH Key (Professional Habit)

### 9.1 Verify Resource Exists in Azure

```bash
az sshkey list -o table
```

Confirmed:

* Name: `devops-kp`
* Location: `westus`
* Resource Group: correct

---

### 9.2 Verify Key Type (RSA)

```bash
ssh-keygen -lf /root/.ssh/1769953461_0779624
```

Output:

```
(RSA)
```

This confirms the cryptographic key type.

---

### 9.3 Verify Using Azure Show Command

```bash
az sshkey show \
  --name devops-kp \
  --resource-group kml_rg_main-9776c0c9b9344238
```

Key fields validated:

* Name
* Location
* Public key starts with `ssh-rsa`

---

## 10. About Location (`--location`) Doubt

Question:

> Do I need to explicitly specify the location?

Answer:

No.

Why?

* Resources inherit the **region of the resource group** by default
* The resource group was already in `westus`

Explicit `--location` is optional unless overriding is required.

---

## 11. Common Beginner Confusions (Cleared)

* ❌ Azure is not confused when it says no public key was provided
* ❌ SSH is not used to *create* VMs — it is used to *access* them
* ❌ DevOps does not require theory-first learning

✔ Verification is as important as creation
✔ Always confirm resource existence

---

## 12. DevOps Mindset Learned

Key takeaways from this single task:

* Secure access comes **before** infrastructure usage
* Always verify what you create
* Cloud tools explain actions transparently
* DevOps is built on small, repeatable habits

> **Create → Verify → Move On**

---

## 13. Final Status

✅ Azure login successful
✅ Resource group identified
✅ SSH key `devops-kp` created
✅ Key type confirmed as RSA
✅ Resource verified via CLI

**Task successfully completed.**

---

## 14. What Comes Next

Likely upcoming tasks:

* Create a VM using `devops-kp`
* SSH into the VM
* Disable password-based authentication
* Automate VM provisioning

This SSH key is now a **foundational asset** for all future DevOps tasks.

---

### End of Documentation
