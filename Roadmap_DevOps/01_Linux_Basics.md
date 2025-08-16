# Linux Foundations for DevOps (Part 1)

When people talk about **DevOps**, the conversation often jumps straight into flashy tools like Docker, Kubernetes, or Terraform. But beneath all of those tools lies a foundation that every DevOps engineer must understand: **Linux**.

Most servers in the cloud run on Linux. Containers are Linux-based. CI/CD pipelines often run in Linux environments. If you’re deploying applications, troubleshooting production issues, or automating workflows, chances are you’ll be working on Linux.  

If you’ve been following my previous blogs, you’ll know that up until now I’ve mostly shared insights around **backend programming, APIs, and system design**. That’s been a big part of my journey so far.  

Now, it’s time to open a **new chapter: DevOps**.  
This will be a new series of technical blogs where I’ll walk through the fundamentals and gradually build up to more advanced topics in the DevOps field.  

And like any good journey, we start with the foundation.  
This article is covering Linux basics — the essential prerequisite for anyone looking to step into the world of DevOps.  


---

## Shells: Bash vs. PowerShell

If you’ve worked on Windows, you’re probably familiar with **Command Prompt** or **PowerShell**. In Linux, the equivalent is **Bash** (Bourne Again Shell). Both are just **command interpreters** that let you interact with the operating system.  

As a DevOps engineer, you’ll often:  
- SSH into remote Linux servers  
- Run Linux-based commands inside containers  
- Use CI/CD pipelines that run shell scripts  

So, knowing Bash is a must-have skill.  

---

## Commonly Used Linux Commands

Here are some of the **most useful commands** that come up almost daily in DevOps work. Think of them as your starter survival kit:  

### Navigation

```bash
pwd        # Print working directory (where am I?)
ls         # List files in the current directory
cd /path   # Change directory
```

### File Operations

```bash
cat file.txt        # Print contents of a file
less file.txt       # View file contents page by page
cp file.txt backup/ # Copy a file
mv file.txt data/   # Move/rename a file
rm file.txt         # Delete a file
```

### Process Management

```bash
ps aux        # Show running processes
top           # Live view of CPU/memory usage
kill <PID>    # Kill a process by ID
```

### Permissions

```bash
chmod +x script.sh      # Make script executable
chown user:user file    # Change file ownership
```

### Networking Basics

```bash
ping google.com          # Check if a host is reachable
curl http://localhost:80 # Make a web request
ss -tulpn                # Show listening ports and processes
```

---

## Package Managers

One of the first things you’ll need to do on a Linux server is **install software**. That’s where **package managers** come in. Instead of downloading installers from websites (like we often do on Windows), Linux uses command-line package managers to fetch and install software from trusted repositories.

Here are the most common ones you’ll see:

- **APT** (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install nginx
````

* **YUM / DNF** (RHEL, CentOS, Fedora)

```bash
sudo yum install httpd
# or, on newer systems
sudo dnf install httpd
```

* **Homebrew** (macOS, bonus for local dev setups)

```bash
brew install wget
```

💡 **Why it matters in DevOps**:
When setting up a CI/CD pipeline, you might need to install dependencies like `dotnet-sdk`, `python3`, or `docker`. Knowing how to use package managers ensures you can script installations reliably.

---

## Services in Linux

In DevOps, many of the applications we deploy run as **services** — background processes managed by the operating system. Think of services like **always-on daemons** (web servers, databases, message queues).

Most modern Linux systems use **systemd** as their service manager. Here are some essential commands:

```bash
# Start a service
sudo systemctl start nginx

# Stop a service
sudo systemctl stop nginx

# Restart a service
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx
```

💡 **Why it matters in DevOps**:
If your web API “isn’t working,” the first thing you might do is check if the service is running. Restarting services, checking logs, and verifying configurations are daily tasks for DevOps engineers.

```bash
curl http://localhost:5000/health
```

---

## VI Editor (Quick Survival Guide)

At some point, you’ll find yourself inside a Linux server where you need to quickly edit a configuration file — maybe a Docker config, an Nginx setting, or an environment variable file. And guess what? The only editor installed might be **VI (or Vim)**.

Here’s a **minimal survival guide** so you don’t panic:

* Open a file:

```bash
vi file.txt
```

* Switch to **insert mode** (to start typing):

```
i
```

* Save and quit:

```
:wq
```

* Quit without saving:

```
:q!
```

That’s enough to survive 95% of real-world cases. If you prefer something friendlier, you can install **nano**, but every DevOps engineer should at least know the basics of VI.

💡 **Pro tip**: When you’re SSH’d into a production server at 2 AM, knowing just these few VI commands can save you from a lot of stress.

---

## Resources for Learning More

Learning Linux can feel overwhelming because it’s such a broad topic. The good news is that as a DevOps engineer, you don’t need to know *everything* — just the essentials for working with servers, containers, and automation.

Here are some resources that helped me:

- **Roadmap.sh – Linux Roadmap**  
  [https://roadmap.sh/linux](https://roadmap.sh/linux)  
  A visual roadmap showing which Linux topics are most relevant for developers and DevOps.

- **YouTube Crash Courses**  
  - [Linux Operating System - Crash Course for Beginners](https://youtu.be/ROjZy1WbCIA?si=Xd1DlSPPGxWzMBY6) (great full walkthrough).  
    - Installing Linux  
    - Desktop environment overview  
    - Terminal basics  
    - Working with directories  
    - Working with files  
    - Viewing and editing file contents  
    - Understanding the Linux file structure  
    - System information commands  
    - Networking basics  
    - Package managers  
    - Text editors  

👉 My advice: **focus only on the parts that matter for DevOps**. Don’t get lost in kernel programming or writing your own drivers. Stick to commands, services, permissions, and the basics of networking.

---

## Conclusion: Linux as the Foundation

In this first part of the DevOps learning journey, I covered the **Linux foundations** that every DevOps engineer should know:

- Navigating with Bash  
- Running essential Linux commands  
- Installing software with package managers  
- Managing services with `systemctl`  
- Surviving in VI editor  

These aren’t just academic exercises — they’re the daily tools you’ll use when deploying apps, managing servers, or debugging pipelines.

💡 The key takeaway: *you don’t need to master all of Linux, but you do need to be comfortable with the essentials*.  

### What’s Next

In next posts, I’ll move into the next layer of DevOps basics:  

- Virtualization (VirtualBox, Vagrant)  
- Networking fundamentals (DNS, IPs, ports)  
- YAML configuration (used everywhere in DevOps)  

Stay tuned! 🚀  

---


