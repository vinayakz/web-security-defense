## INVESTIGATION COMMANDS USED FOR SECURITY ANALYSIS
  ====== SYSTEM STATUS COMMANDS ======

```bash
# Check Apache2 service status
systemctl status apache2

# Check MySQL/MariaDB service status  
systemctl status mysql

# Check current memory usage
free -h

# Check swap configuration
swapon --show

```
====== LOG ANALYSIS COMMANDS ======
```bash
 

# Check Apache2 logs for shutdown events from yesterday
journalctl -u apache2 --since "yesterday" --until "today" | grep -E "(stop|fail|error|kill|shutdown)" | tail -20

# Check MySQL logs for shutdown events from yesterday
journalctl -u mariadb --since "yesterday" --until "today" | grep -E "(stop|fail|error|kill|shutdown)" | tail -20

# Check system logs for OOM killer events
journalctl --since "2026-06-14 00:00" --until "2026-06-18 06:00" | grep -E "(Out of memory|OOM|killed|oom-killer)" | tail -10

# Check kernel messages for OOM events
sudo dmesg | grep -E "(Out of memory|OOM|killed)" | tail -10

```
 ====== MEMORY ANALYSIS COMMANDS ======
```bash


# Check memory configuration
cat /proc/sys/vm/swappiness
cat /proc/sys/vm/overcommit_memory

# Check current memory-intensive processes
ps aux --sort=-%mem | head -10

```
====== ATTACK DETECTION COMMANDS ======
```bash
  
# Check recent Apache access logs
sudo tail -50 /var/log/apache2/access.log | grep -E "(POST|GET)" | head -10

# Check for suspicious HTTP errors and POST requests
sudo grep -E "(404|500|POST)" /var/log/apache2/access.log | tail -20 | grep -v "wp-cron\|wp-json"

# Count XML-RPC attack attempts
sudo grep -c "xmlrpc.php" /var/log/apache2/access.log

# Check recent XML-RPC attacks
sudo grep "xmlrpc.php" /var/log/apache2/access.log | tail -10

# Check auth logs for attack attempts
sudo grep -E "(brute|attack|hack|exploit)" /var/log/auth.log | tail -5
```
====== NETWORK ANALYSIS COMMANDS ======
```bash
 
# Check listening ports
sudo netstat -tuln | grep :80

# Check for scheduled tasks
crontab -l 2>/dev/null || echo "No crontab found"

# Check system-wide cron jobs
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ /etc/cron.weekly/ 2>/dev/null | grep -v "^total"
```
====== DETAILED LOG EXTRACTION COMMANDS ======
```bash

# Get detailed OOM killer context
sudo journalctl --since "2026-05-29 01:25" --until "2026-05-29 01:35" | grep -A5 -B5 "oom-killer"

# Extract XML-RPC attack patterns
sudo grep "xmlrpc.php" /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -20

# Check for other attack vectors
sudo grep -E "(\.env|wp-admin|wp-login)" /var/log/apache2/access.log | tail -10
```

====== SYSTEM CONFIGURATION COMMANDS ======
```bash
# Check Apache configuration
sudo apache2ctl -S

# Check MySQL configuration
sudo mysql -e "SHOW VARIABLES LIKE '%buffer%';" 2>/dev/null || echo "MySQL connection failed"

# Check system limits
ulimit -a

# Check disk usage
df -h
```
====== REAL-TIME MONITORING COMMANDS ======
```bash

# Monitor real-time attacks (run for 30 seconds)
 sudo tail -f /var/log/apache2/access.log | grep xmlrpc

# Monitor memory usage in real-time
 watch -n 1 'free -h && ps aux --sort=-%mem | head -5'

# Monitor system load
uptime
```

====== SECURITY HARDENING VERIFICATION ======
```bash

# Check if fail2ban is installed
which fail2ban-server || echo "Fail2ban not installed"

# Check firewall status
sudo ufw status || echo "UFW not configured"

# Check for security updates
apt list --upgradable 2>/dev/null | grep -i security | wc -l
```

 ====== END OF INVESTIGATION COMMANDS ======
