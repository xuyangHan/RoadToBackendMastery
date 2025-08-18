# Virtualization, Networking, and YAML Basics

## 1. Introduction

In [the previous blog](02_Linux_Basics.md), we explored the **Linux basics** every DevOps engineer needs: navigating with Bash, managing services, and installing software with package managers.  

Now, it’s time to take the next step in the journey. To truly understand DevOps, you need to know more than just commands inside a single machine — you need to understand how **environments are created, how they communicate, and how we configure them consistently**.  

In this article, we’ll cover three important building blocks:  

- **Virtualization** → running environments safely and consistently.  
- **Networking** → how systems talk to each other.  
- **YAML** → the configuration language gluing modern DevOps tools together.  

---

## 2. Virtualization Basics

### What is Virtualization?

At its core, **virtualization** means running “a computer inside your computer.”  
Instead of needing multiple physical servers, virtualization lets you create virtual ones on the same hardware.  

Why does this matter for DevOps?  

- You can spin up **repeatable test environments** without extra hardware.  
- You can run **safe experiments** — break things in a VM without harming your laptop.  
- It’s the foundation for **cloud computing**, where companies run thousands of virtual machines at scale.  

---

### Starters Virtualization Tools

- [**VirtualBox**](https://www.virtualbox.org/)
  A free and popular tool for running virtual machines locally. Great for beginners who want to practice Linux without dual-booting or buying extra hardware.  

- [**Vagrant**](https://developer.hashicorp.com/vagrant)
  A companion tool that automates VM setup. With a single config file, you can spin up a VM with predefined settings — making environments **reproducible** across teams.  

💡 *Learning tip*: VirtualBox + Vagrant is perfect for local practice, especially when you’re just starting out.  

---

### Virtualization in the Industry

While VirtualBox and Vagrant are useful for learning, modern companies usually rely on:  

- **Cloud VMs** like AWS EC2, Azure VMs, and Google Cloud Compute Engine.  
- **Containers** (Docker) for lightweight, fast, and portable environments.  

So think of VirtualBox as your **training ground**, and cloud/containers as the **real-world application** of virtualization concepts.  

---

### VMs vs Containers

- **Virtual Machines (VMs)**:  
  - Each VM runs a full operating system with its own kernel.  
  - More isolated but heavier on resources (RAM, CPU).  
  - Example: Running Ubuntu and Windows side-by-side on your laptop.  

- **Containers (Docker)**:  
  - Share the host operating system’s kernel.  
  - Lightweight, fast to start, and use fewer resources.  
  - Example: Running an API inside a container, isolated from other apps.  

Both approaches aim to provide **isolated environments**, but they solve different problems.  

---

➡️ *More to know: If you’ve been following my **Backend Programming Roadmap**, you might remember I already wrote about [**containers and virtualization**](../Roadmap_Backend/14_Container.md) from a backend perspective. That article focused more on how containers are used in software development.*

---

## 3. Networking Fundamentals

When we move from a single computer to multiple systems, the key question becomes: **how do they talk to each other?**  
That’s where networking concepts come in. As a DevOps engineer, you don’t need to be a network engineer, but you do need to understand the basics to debug and deploy systems.

### DNS (Domain Name System)

- DNS is like the internet’s phonebook.  
- It translates human-friendly names like `google.com` into machine-friendly IP addresses like `142.250.191.46`.  
- Why it matters in DevOps: if DNS is misconfigured, your services won’t be reachable, even if the servers are running fine.

### IP Addresses

- An **IP address** identifies a machine on a network.  
- **IPv4 basics**: looks like `192.168.0.1`.  
- **Private IPs**: used inside networks (e.g., home WiFi or cloud VPCs).  
- **Public IPs**: visible to the internet.  

💡 **Tip**: In the cloud, a single VM may have both a private IP (internal communication) and a public IP (external access).  

### Ports

- One machine (IP) can run many services at once. **Ports** separate them.  
- Common ports:  
  - 80 → HTTP (websites)  
  - 443 → HTTPS (secure websites)  
  - 5432 → PostgreSQL  
  - 6379 → Redis  

Example: If you run a web API on port 8080, you can test it with:  

```bash
curl http://localhost:8080
```

If it responds, the service is working. If not, it could be a networking issue (firewall, DNS, wrong port).

---

➡️ *More to know: If you’ve been following my **Backend Programming Roadmap**, you might remember I already wrote about [**networking and the internet**](../Roadmap_Backend/01_Internet.md) from a backend perspective. That article explained how the web works at the protocol level (HTTP/HTTPS, requests, and responses).*

---

## 4. YAML Basics

Modern DevOps tools rely heavily on **configuration files**. Instead of hardcoding settings or using messy scripts, we define infrastructure and workflows using config languages. And YAML has become the most popular choice.

### Why YAML?

* Used everywhere: **Docker Compose, Kubernetes, GitHub Actions, Ansible**, and more.
* Human-readable, indentation-based, and cleaner than JSON or XML for configs.

### Example

Messy JSON:

```json
{"service": {"name":"api","replicas":3,"port":8080}}
```

Clean YAML:

```yaml
service:
  name: api
  replicas: 3
  port: 8080
```

### Gotchas

* YAML is **space-sensitive**. Wrong indentation = broken config.
* Tabs are not allowed, only spaces.

💡 **Pro tip**: Always use a YAML-aware code editor (VS Code, IntelliJ, etc.) to avoid invisible formatting errors.

---

## 5. Lessons Learned

* **Virtualization** helps us simulate real servers locally, and it scales all the way to the cloud.
* **Networking** is the language of communication between systems — even basic knowledge saves hours of debugging.
* **YAML** may look boring, but it’s the glue that holds modern DevOps tools together.

These might feel like small building blocks, but they form the **foundation for everything in DevOps**. Without them, tools like Docker, Kubernetes, or Terraform would be much harder to understand.

---

## 6. What’s Next

In the next article, we’ll build on this foundation and take the first step into **automation**:

* Setting up a simple **CI/CD pipeline** to automate builds, tests, and deployments.

Stay tuned — this is where DevOps starts to feel magical 🚀.
