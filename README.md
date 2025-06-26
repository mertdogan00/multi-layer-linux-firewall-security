# 🛡️ Linux Server Security: Layered Protection (firewalld, UFW, iptables, OSI) 🚀

A comprehensive and beginner-friendly guide for securing Linux servers using **firewalld**, **UFW**, and **iptables** with explanations, live test commands, and detailed OSI model references. Secure your server step by step, with clarity on what every command and rule does!

---

## 📚 Table of Contents

* [Introduction](#introduction)
* [Understanding the OSI Model](#understanding-the-osi-model)
* [Why Firewall Layers?](#why-firewall-layers)
* [Firewall Tools Overview: firewalld, UFW, iptables](#firewall-tools-overview-firewalld-ufw-iptables)
* [Initial Server Setup](#initial-server-setup)
* [firewalld Configuration](#firewalld-configuration)
* [UFW Configuration](#ufw-configuration)
* [iptables: Basics and Usage](#iptables-basics-and-usage)
* [Testing & Simulating Attacks](#testing--simulating-attacks)
* [Advanced Tips](#advanced-tips)
* [FAQ](#faq)
* [Contributing](#contributing)
* [License](#license)

---

## Introduction

Linux servers are exposed to the internet and are regular targets of automated and targeted attacks. Using multiple layers of firewalls (software, network, service) ensures much better protection than relying on defaults.

This guide covers **firewalld**, **UFW (Uncomplicated Firewall)**, and **iptables** – the most common Linux firewall tools. All steps are explained for clarity and security best practices.

---

## Understanding the OSI Model

Firewalls operate at different layers of the [OSI model](https://en.wikipedia.org/wiki/OSI_model):

1. **Layer 3 (Network)** – Controls IP traffic (e.g., ICMP, ping, IP block)
2. **Layer 4 (Transport)** – Controls TCP/UDP ports (e.g., block SSH brute force, open HTTP/HTTPS)
3. **Layer 7 (Application)** – Controls by application protocol (e.g., HTTP flood, rate limit)

**Knowing what happens at each layer is critical!**

| Layer | Example Attack/Test      | Firewall Handles?   |
| ----- | ------------------------ | ------------------- |
| 3     | `ping`, ICMP flood       | firewalld, iptables |
| 4     | SYN flood, TCP port scan | firewalld, iptables |
| 7     | HTTP flood               | Nginx/Apache, WAF   |

---

## Why Firewall Layers?

* **Multiple tools = multiple barriers.**
* If an attacker bypasses one, others may still block them.
* Allows separation of duties and granular control.

**Analogy:** Like locking your door, then setting an alarm, then putting bars on the window – layers make it harder to break in.

---

## Firewall Tools Overview: firewalld, UFW, iptables

### 🔥 firewalld

* **Modern** firewall manager for Linux (uses zones and services).
* Dynamically manages rules without restarts.
* Works as an abstraction layer over iptables/nftables.
* **Good for:** Servers, desktops, easy zone management.

### 🔥 UFW (Uncomplicated Firewall)

* Very simple and user-friendly.
* Works as a wrapper around iptables.
* **Good for:** Beginners, Ubuntu users.

### 🔥 iptables

* Low-level tool, direct rule writing.
* Powerful but requires more expertise.
* **Good for:** Power users, automation, complex filtering.

You can use **one**, **two**, or all three – but avoid conflicting rules.

---

## Initial Server Setup

Always start by **updating your packages**:

```bash
sudo apt update && sudo apt upgrade -y      # Debian/Ubuntu
sudo dnf update -y                          # CentOS/RHEL/Fedora
```

Check which firewall tools are installed:

```bash
which firewall-cmd   # firewalld
which ufw            # UFW
which iptables       # iptables
```

---

## firewalld Configuration

### 1. Check Default Zone

```bash
firewall-cmd --get-default-zone
```

The default is usually `public`.

### 2. Harden by setting default to `drop`

```bash
sudo firewall-cmd --set-default-zone=drop
```

This blocks all traffic unless specifically allowed.

### 3. Assign interface to a zone (example: `eth0`)

```bash
sudo firewall-cmd --permanent --zone=public --add-interface=eth0
```

### 4. Check and set zone target

```bash
firewall-cmd --zone=public --get-target --permanent
sudo firewall-cmd --permanent --zone=public --set-target=DROP
```

### 5. Allow necessary ports (example: HTTP, HTTPS, custom SSH)

```bash
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp      # HTTP
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp     # HTTPS
sudo firewall-cmd --permanent --zone=public --add-port=2222/tcp   # Custom SSH
```

### 6. Reload to apply

```bash
sudo firewall-cmd --reload
```

### 7. Verify settings

```bash
firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --list-all
```

#### **What does each step do?**

* **Default zone drop:** All traffic is blocked unless allowed.
* **Assign interface:** Which network card the rule applies to.
* **Allow ports:** Opens only needed ports to the world.
* **Reload:** Makes settings active immediately.

---

## UFW Configuration

### 1. Enable UFW

```bash
sudo ufw enable
```

### 2. Allow necessary ports

```bash
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow 56470/tcp   # Custom SSH
```

### 3. Check rules

```bash
sudo ufw status verbose
```

#### **What does UFW do?**

* Default is to **deny all incoming** traffic.
* Each `allow` opens a specific port for access.
* Simple, readable syntax for daily use.

---

## iptables: Basics and Usage

### 1. Check current rules

```bash
sudo iptables -L -n -v
```

### 2. Block all incoming traffic by default

```bash
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

### 3. Allow established sessions

```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### 4. Allow SSH (custom port example: 56470)

```bash
sudo iptables -A INPUT -p tcp --dport 56470 -j ACCEPT
```

### 5. Allow HTTP/HTTPS

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

### 6. Save and persist rules

* On Ubuntu: `sudo iptables-save > /etc/iptables/rules.v4`
* On CentOS: use `service iptables save` or use `iptables-save`

#### **What is iptables?**

* Works at Layer 3/4.
* Direct packet and connection rules.
* Very flexible, but can break connectivity if misconfigured!

---

## Testing & Simulating Attacks

**Layer 3 (ICMP/ping):**

```bash
ping [your-server-ip]
```

* 100% packet loss: your firewall drops ICMP requests.

**Layer 4 (TCP SYN Flood):**

```bash
sudo hping3 -S --flood -p 443 [your-server-ip]
```

* No server response = TCP flood blocked.

**Layer 7 (HTTP flood):**

```bash
seq 10000 | parallel -j200 curl -s http://[your-server-ip]/ > /dev/null
```

* Requests reach web server, shown in logs (application firewall/WAF needed to block here).

**Increase file limit for testing:**

```bash
ulimit -n 65535
```

---

## Advanced Tips

* **Combine firewalld/UFW with fail2ban** for extra brute-force protection.
* Use **cloud provider security groups** (e.g. AWS, DigitalOcean) as a first layer.
* Always use **custom SSH ports** and key-based authentication.
* Backup firewall configs regularly.
* Test rules in a staging VM before applying to production!

---

## FAQ

**Q: Should I run firewalld and UFW together?**

* *Usually not needed.* Choose one main firewall manager to avoid conflicts.

**Q: Is iptables still necessary?**

* *Yes*, for advanced users and automation. Most modern tools use iptables/nftables in the background.

**Q: How do I reset all firewall rules?**

* firewalld: `sudo firewall-cmd --complete-reload`
* ufw: `sudo ufw reset`
* iptables: `sudo iptables -F`

---

## Contributing

Pull requests, improvements, and real-world test results are very welcome! Please open an issue or submit a PR.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for full details.
