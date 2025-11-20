<div align="center">

# 🛡️ iptables Management Script

**A safe, simple tool for managing iptables firewall rules with automatic backups and rollback protection to prevent lockouts.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Shell Script](https://img.shields.io/badge/Shell_Script-Bash-green.svg)](https://www.gnu.org/software/bash/)
[![Platform](https://img.shields.io/badge/platform-Linux-blue.svg)](https://www.kernel.org/)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/NexteraMatt/iptables-management-script/graphs/commit-activity)

[Features](#-features) • [Installation](#-installation) • [Usage](#-usage) • [Safety Features](#-safety-features)

</div>

---

## ⚠️ The Problem

Managing iptables rules on production servers is risky:

| Risk | Consequence |
|------|-------------|
| 🔒 **Lockout risk** | One wrong rule = permanent server lockout |
| ⏮️ **No undo** | Mistakes are difficult/impossible to reverse remotely |
| 💾 **Manual backups** | Easy to forget, error-prone |
| 🤯 **Complex syntax** | iptables commands are verbose and confusing |
| 🧪 **Testing danger** | Hard to safely test changes in production |

---

## ✨ The Solution

This script provides a safe wrapper around iptables with:

```
┌─────────────────────┐
│  Your Command       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Auto Backup        │  ← Creates timestamped backup
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Validation         │  ← Checks syntax
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Apply Changes      │  ← Makes changes
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Logging            │  ← Audit trail
└─────────────────────┘
```

---

## 🚀 Features

<table>
<tr>
<td width="50%">

### Safety First
- 🛡️ **Safe by default** - Auto backup before changes
- 🔄 **Rollback support** - Revert to previous config
- ⏲️ **Timed changes** - Auto-revert if connection lost
- ✅ **Validation** - Syntax checking before applying

</td>
<td width="50%">

### Ease of Use
- 🎚️ **Rule toggles** - Enable/disable without deletion
- 📝 **Change logging** - Track all modifications
- 🔍 **Rule inspection** - Easy viewing of rules
- 💾 **Backup history** - Timestamped backups

</td>
</tr>
</table>

---

## 📋 Prerequisites

```bash
✓ Linux server with iptables
✓ Root or sudo access
✓ Bash 4.0 or higher
```

---

## 🔧 Installation

**1️⃣ Clone the repository**
**1️⃣ Clone the repository**
```bash
git clone https://github.com/NexteraMatt/iptables-management-script.git
cd iptables-management-script
```

**2️⃣ Make the script executable**
```bash
chmod +x iptables-manager.sh
```

**3️⃣ Optionally, copy to your PATH**
```bash
sudo cp iptables-manager.sh /usr/local/bin/iptables-manager
```

---

## 🎮 Usage

### Basic Operations

**Add a new rule (with automatic backup):**
```bash
sudo ./iptables-manager.sh add -p tcp --dport 8080 -j ACCEPT
```

**View current rules:**
```bash
sudo ./iptables-manager.sh list
```

**Backup current rules:**
```bash
sudo ./iptables-manager.sh backup
```

**Restore from backup:**
```bash
sudo ./iptables-manager.sh restore
```

### Safe Remote Changes

**Test a change with auto-rollback (useful for remote servers):**
```bash
# Apply rule and auto-revert after 60 seconds if not confirmed
sudo ./iptables-manager.sh add -p tcp --dport 22 -j ACCEPT --test-mode 60

# If connection still works, confirm the change:
sudo ./iptables-manager.sh confirm
```

### Rule Management

**Toggle rule on/off without deletion:**
```bash
# Disable a rule temporarily
sudo ./iptables-manager.sh toggle <rule_number> off

# Re-enable it later
sudo ./iptables-manager.sh toggle <rule_number> on
```

**View backup history:**
```bash
sudo ./iptables-manager.sh list-backups
```

---

## 🔒 Safety Features

<table>
<tr>
<td width="50%">

### 💾 Automatic Backup
Every operation creates a timestamped backup automatically.

**Backup Location:** `/etc/iptables/backups/`

```
📁 /etc/iptables/backups/
├── 📄 backup-2025-01-15-14-30-22.rules
├── 📄 backup-2025-01-15-15-45-10.rules
└── 🔗 latest → backup-2025-01-15-15-45-10.rules
```

</td>
<td width="50%">

### 🧪 Test Mode
Apply changes with automatic rollback:

```bash
sudo ./iptables-manager.sh add <rule> \
  --test-mode 120
# Auto-revert after 120 seconds 
# unless confirmed
```

**Perfect for remote changes!**

</td>
</tr>
<tr>
<td width="50%">

### ✅ Validation
All rules validated for syntax errors before applying.

**Catches:**
- Malformed rules
- Invalid port numbers
- Missing required flags
- Incompatible options

</td>
<td width="50%">

### 📝 Change Logging
All modifications logged with:

- ⏰ Timestamp
- 👤 User who made change
- 🔧 Exact command executed
- 📊 Before/after rule count

**Log:** `/var/log/iptables-manager.log`

</td>
</tr>
</table>

---

## 📚 Common Use Cases

### Opening a New Port
```bash
# Allow incoming traffic on port 443
sudo ./iptables-manager.sh add -A INPUT -p tcp --dport 443 -j ACCEPT
```

### Restricting Access by IP
```bash
# Allow SSH only from specific IP
sudo ./iptables-manager.sh add -A INPUT -p tcp --dport 22 -s 192.168.1.100 -j ACCEPT
```

### Temporarily Blocking Traffic
```bash
# Block all traffic from an IP (with easy toggle back)
sudo ./iptables-manager.sh add -A INPUT -s 10.0.0.5 -j DROP
# Later: toggle it off instead of deleting
sudo ./iptables-manager.sh toggle 5 off
```

### Remote Server Updates
```bash
# Safest way to update rules on a remote server
sudo ./iptables-manager.sh add -A INPUT -p tcp --dport 2222 -j ACCEPT --test-mode 300
# Test your connection on the new port
# If it works:
sudo ./iptables-manager.sh confirm
# If it fails, rules auto-revert after 5 minutes
```

## Why This Script Exists

As a sysadmin managing hundreds of Linux servers, I needed a way to safely modify firewall rules without the constant fear of locking myself out. This script has prevented countless lockout situations and saved hours of emergency datacenter trips.

## Best Practices

1. **Always use test mode for remote changes** - Better safe than making an emergency datacenter visit
2. **Review rules before confirming** - The test period is your chance to catch mistakes
3. **Keep regular backups** - Even though the script backs up automatically, periodic manual backups are good practice
4. **Document your rules** - Use comments in your iptables rules to explain their purpose
---

## 💡 Best Practices

> **🎯 Pro Tips from Production Experience**

| Practice | Why It Matters |
|----------|----------------|
| ⏲️ **Always use test mode for remote changes** | Better safe than an emergency datacenter visit |
| 👀 **Review rules before confirming** | The test period is your chance to catch mistakes |
| 💾 **Keep regular backups** | Extra insurance beyond automatic backups |
| 📝 **Document your rules** | Use comments to explain purpose |
| 🧪 **Test in staging first** | Catch issues before production |

---

## 🛠️ Stack

<p align="center">
  <img src="https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash" />
  <img src="https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux" />
  <img src="https://img.shields.io/badge/iptables-Firewall-red?style=for-the-badge" alt="iptables" />
</p>

---

## 🤝 Contributing

Contributions are welcome! Please:

- ✅ Test changes on non-production systems first
- ✅ Include examples for new features
- ✅ Update documentation
- ✅ Follow existing code style

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📬 Support

<div align="center">

**Need help or found a bug?**

[![GitHub Issues](https://img.shields.io/github/issues/NexteraMatt/iptables-management-script)](https://github.com/NexteraMatt/iptables-management-script/issues)

[Open an Issue](https://github.com/NexteraMatt/iptables-management-script/issues) • [Visit Portfolio](https://matthodges.uk)

</div>

---

## ⚠️ Safety Warning

<div align="center">

**⚠️ IMPORTANT ⚠️**

**Always be cautious when modifying firewall rules on production systems.**

Even with safety features, incorrect rules can cause service disruption.

**Test in non-production environments when possible.**

</div>

---

<div align="center">

**Made with ❤️ by [Matt Hodges](https://matthodges.uk)**

*This script has prevented countless server lockouts and saved hours of emergency datacenter trips.*

⭐ Star this repo if it saves you from a lockout!

</div>
