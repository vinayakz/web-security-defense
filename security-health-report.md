# Security & Health Check Report
**Date:** 2026-07-15

---

## 1. SSH Brute Force Attack

**Problem:** 131 SSH login attempts from `50.220.35.122` on Jul 13 (07:08–08:04) using common usernames like `admin`, `ghost`, `kafka`, `ftpuser`, etc.

**Check command:**
```bash
sudo lastb | head -20
sudo grep -i "failed\|invalid" /var/log/auth.log | tail -20
sudo grep -i "50.220.35.122" /var/log/auth.log | wc -l
```

**Result:** All attempts failed. No unauthorized access.

---

## 2. SSH Port Open to Everyone (UFW Misconfiguration)

**Problem:** Port 22 was open to `Anywhere` instead of only your IP.

**Check command:**
```bash
sudo ufw status verbose
```

**Fix applied:**
```bash
sudo ufw delete allow 22/tcp
sudo ufw allow from 27.107.88.190 to any port 22
```

**Result:** Port 22 now restricted to `27.101.18.180` only. ✅

---

## 3. fail2ban Not Active

**Problem:** fail2ban was installed but the sshd jail was not enabled — `jail.local` was missing, so no IPs were ever banned despite 131 brute force attempts.

**Check command:**
```bash
sudo fail2ban-client status sshd
```

**Fix applied:**
```bash
sudo bash -c 'cat > /etc/fail2ban/jail.local << EOF
[sshd]
enabled = true
maxretry = 5
bantime = 1h
findtime = 10m
EOF'
sudo systemctl restart fail2ban
```

**Result:** fail2ban running with sshd jail active. Auto-bans after 5 failed attempts. ✅

---

## 4. World-Writable File Permissions

**Problem:** `/etc`, `/etc/cron.d`, `/etc/crontab`, and all cron directories had `rwxrwxrwx` permissions — any user could modify them.

**Check command:**
```bash
ls -la /etc/cron*
```

**Fix:**
```bash
sudo chmod 644 /etc/crontab
sudo chmod 755 /etc/cron.d /etc/cron.daily /etc/cron.hourly /etc/cron.weekly /etc/cron.monthly
sudo chown root:root /etc /etc/crontab /etc/cron.d
```

---

## 5. phpMyAdmin Brute Force

**Problem:** Multiple unauthorized login attempts to phpMyAdmin on Jul 10 from IPs: `93.161.7.17`, `5.128.204.57`, `186.93.76.153`, `201.158.40.8`, etc.

**Check command:**
```bash
sudo grep "user denied" /var/log/syslog | tail -20
```

**Fix:**
```bash
sudo nano /etc/apache2/conf-available/phpmyadmin.conf
# Add inside <Directory> block:
# Require ip 27.107.88.190
sudo systemctl reload apache2
```

---

## 6. Website (Apache) Status

**Check command:**
```bash
sudo systemctl status apache2 --no-pager
curl -sk -o /dev/null -w "%{http_code}" http://localhost
```

**Result:** Running healthy for 6 days. HTTP responding. ✅

---

## 7. Database (MariaDB) Status

**Check command:**
```bash
sudo systemctl status mariadb --no-pager
sudo mysqladmin ping
```

**Result:** Running healthy since Jul 13. `mysqld is alive`. ✅

---

## 8. Disk Usage Warning

**Problem:** Disk at 73% used (36GB / 49GB). Not critical but needs monitoring.

**Check command:**
```bash
df -h /
```

**Fix:**
```bash
sudo journalctl --vacuum-size=200M
sudo apt autoremove -y && sudo apt clean
```

---

## 9. System Resources

**Check command:**
```bash
free -h && uptime
```

| Resource | Status |
|----------|--------|
| RAM | 1GB used / 7.7GB total ✅ |
| Swap | 33MB used / 2GB total ✅ |
| Load avg | 0.07 ✅ |
| Uptime | 166 days ✅ |

---

## Summary

| # | Issue | Severity | Status |
|---|-------|----------|--------|
| 1 | SSH brute force from `50.220.35.122` | 🔴 High | No breach, monitored ✅ |
| 2 | SSH port open to everyone | 🔴 High | Fixed ✅ |
| 3 | fail2ban not banning | 🔴 High | Fixed ✅ |
| 4 | World-writable /etc & cron | 🔴 High | Needs fix ⚠️ |
| 5 | phpMyAdmin brute force | 🟠 Medium | Needs fix ⚠️ |
| 6 | Apache website | 🟢 Healthy | Running ✅ |
| 7 | MariaDB database | 🟢 Healthy | Running ✅ |
| 8 | Disk at 73% | 🟡 Low | Monitor ⚠️ |
| 9 | System resources | 🟢 Healthy | Normal ✅ |
