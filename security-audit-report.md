# Security Audit Report — July 13, 2026

## 🔴 ACTIVE THREATS

### 1. XML-RPC Brute Force Attack (ONGOING — HIGH)
- IP `168.197.208.249` hammering `/xmlrpc.php` every ~15 seconds with rotating User-Agents
- 22+ requests in last hour — all returning HTTP 200 (endpoint is open)
- Goal: credential brute-force / botnet amplification

### 2. SSH Brute Force Attacks (ONGOING — HIGH)
| IP | Failed Attempts |
|---|---|
| 172.31.3.106 | 233,711 |
| 82.151.65.155 | 2,092 |
| 102.244.97.186 | 1,921 |
| 4.4.66.82 | 1,340 |
| 136.232.70.150 | 1,150 |

- Usernames tried: `root`, `eduardo`, `jack`, `whatsapp`, `fastuser`
- All attempts failed — no unauthorized logins

### 3. fail2ban Installed but NOT Running
- Auto-banning is disabled — brute-force IPs are not being blocked

### 4. Sitecore Probe Attempts
- IPs `45.156.129.137`, `45.156.128.168` probing `/sitecore/shell/sitecore.version.xml`
- Known scanner looking for Sitecore CMS vulnerabilities

---

## 🟡 SUSPICIOUS FILES

| File | Status |
|---|---|
| `/var/tmp/wp-config.php.swp` | Vim swap file — unclosed edit session, low risk |
| `/wp-content/uploads/sucuri/*.php` | Legitimate (Sucuri plugin) |
| `/wp-content/uploads/redux/*.php` | Legitimate (Redux Framework) |
| `/wp-content/uploads/wpseo-redirects/index.php` | Legitimate (Yoast SEO) |

No malicious code (eval/base64/shell) found in any files.

---

## 🟢 CLEAN

- No unauthorized successful SSH logins
- No webshells found
- No malicious processes running
- No suspicious cron jobs
- Ports 22, 80, 443 open; 3306 localhost only
- Only authorized IPs `27.107.88.190` and `103.81.36.93` logged in successfully

---

## Solutions

### Step 1 — Block Active Attackers
```bash
sudo ufw enable
sudo ufw deny from 168.197.208.249
sudo ufw deny from 172.31.3.106
```

### Step 2 — Disable xmlrpc.php
```bash
echo '<Files xmlrpc.php>
Order Deny,Allow
Deny from all
</Files>' | sudo tee -a /var/www/html/.htaccess
```

### Step 3 — Start fail2ban
```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### Step 4 — Configure fail2ban
```bash
sudo tee /etc/fail2ban/jail.local <<EOF
[sshd]
enabled = true
maxretry = 5
bantime = 3600
findtime = 600

[apache-auth]
enabled = true
maxretry = 5
bantime = 3600
EOF
sudo systemctl restart fail2ban
```

### Step 5 — Harden SSH
```bash
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### Step 6 — Remove Swap File
```bash
rm /var/tmp/wp-config.php.swp
```

### Step 7 — Enable UFW with Rate Limiting
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw limit 22/tcp
sudo ufw enable
```

---

## Summary

| Area | Status |
|---|---|
| Unauthorized access | ✅ None found |
| SSH brute force | 🔴 Active — fail2ban not running |
| XML-RPC attack | 🔴 Active — needs blocking |
| Webshells | ✅ None found |
| Malicious processes | ✅ None found |
| Suspicious files | 🟡 1 swap file (low risk) |
| Firewall (UFW) | 🔴 Not active |


-------------------------------------------------------------------------------------------------

# Server Security Setup Report

## 1. MySQL Port (3306) — Security Group Check

| Port | Protocol | Source | Status |
|------|----------|--------|--------|
| 22 (SSH) | TCP | 0.0.0.0/0 | ⚠️ Open to all |
| 80 (HTTP) | TCP | 0.0.0.0/0 | ✅ Expected |
| 443 (HTTPS) | TCP | 0.0.0.0/0 | ✅ Expected |
| 8080 | TCP | 0.0.0.0/0 | ⚠️ Open to all |
| 3306 (MySQL) | TCP | 27.107.88.190/32 | ✅ Your IP only |

- MySQL port 3306 is restricted to your IP only (`/32` = single IP) ✅
- SSH port 22 is open to the world — recommended to restrict to your IP

---

## 2. Fail2ban Setup

- Status before: installed but inactive ❌
- Actions taken:
  ```bash
  sudo systemctl start fail2ban
  sudo systemctl enable fail2ban
  ```
- Status after: ✅ active and enabled on boot
- Active jail: `sshd` — brute force SSH protection is ON
- Website impact: none

---

## 3. UFW (Firewall) Setup

### 3.1 Fixed File Permissions
All UFW config files were world-writable. Fixed with:
```bash
sudo chmod 644 /etc/ufw/*.rules /etc/ufw/ufw.conf /etc/ufw/applications.d/*
sudo chmod 755 /etc/ufw /etc/ufw/applications.d
sudo chown root:root /etc/default/ufw && sudo chmod 644 /etc/default/ufw
```

### 3.2 UFW Rules Added
```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8080/tcp
sudo ufw allow from 27.107.88.190 to any port 3306
sudo ufw --force enable
```

### 3.3 Final UFW Rules

| # | Port | Action | From |
|---|------|--------|------|
| 1 | 22/tcp (SSH) | ALLOW | Anywhere |
| 2 | 80/tcp (HTTP) | ALLOW | Anywhere |
| 3 | 443/tcp (HTTPS) | ALLOW | Anywhere |
| 4 | 8080/tcp | ALLOW | Anywhere |
| 5 | 3306 (MySQL) | ALLOW | 27.107.88.190 only |

---

## 4. MySQL Config File Permissions Fix

All `/etc/mysql/*.cnf` files were world-writable. Fixed with:
```bash
sudo find /etc/mysql -type f -name "*.cnf" | xargs sudo chmod 644
sudo find /etc/mysql -type f -name "*.cnf" | xargs sudo chown root:root
```

Files fixed:
- `/etc/mysql/conf.d/mysql.cnf`
- `/etc/mysql/conf.d/mysqldump.cnf`
- `/etc/mysql/mariadb.conf.d/50-client.cnf`
- All other `.cnf` files under `/etc/mysql/`

---

## 5. Final Verification

| Service | Status |
|---------|--------|
| UFW Firewall | ✅ Active & enabled on boot |
| Fail2ban | ✅ Active & enabled on boot |
| Website HTTP (port 80) | ✅ 302 responding |
| Website HTTPS (port 443) | ✅ 301 responding |
| MySQL/MariaDB | ✅ Running |
| MySQL config permissions | ✅ Secured (root owned, 644) |