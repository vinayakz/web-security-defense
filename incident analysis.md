# WordPress Security Incident Investigation: Linux Commands Every DevOps Engineer Should Know

## Introduction

When a production website suddenly starts crashing, consuming excessive memory, or becoming unavailable, the first responsibility of a DevOps Engineer is to identify the root cause quickly.

Recently, I investigated a WordPress server experiencing:

- High memory usage
- Apache crashes
- MariaDB OOM kills
- Website downtime
- Massive XML-RPC attack traffic

This article explains the investigation process and the Linux commands I used to identify the issue.

---

## Step 1: Verify Service Status

Before investigating logs, check whether critical services are running.

## Apache Status

```bash
systemctl status apache2
```

Purpose:

- Check current state
- Verify restart history
- Review recent failures

---

## MariaDB Status

```bash
systemctl status mysql
```

or

```bash
systemctl status mariadb
```

Purpose:

- Confirm database availability
- Detect unexpected restarts

---

## Step 2: Analyze Memory Usage

Since the server was crashing, memory was my first focus.

## Check Current Memory Usage

```bash
free -h
```

Example:

```text
Mem: 7.8G
Used: 7.2G
Free: 0.2G
```

Purpose:

- Detect memory exhaustion
- Check available RAM

---

## Verify Swap Usage

```bash
swapon --show
```

Purpose:

- Confirm swap availability
- Determine whether the system can absorb memory spikes

---

# Step 3: Search for OOM Killer Events

The Linux kernel automatically kills processes when memory becomes exhausted.

## System Journal

```bash
journalctl --since "2026-05-29 00:00" --until "2026-05-30 06:00" | grep -E "(Out of memory|OOM|killed|oom-killer)"
```

---

## Kernel Messages

```bash
dmesg | grep -E "(Out of memory|OOM|killed)"
```

Example:

```text
Out of memory: Kill process 2345 (apache2)
Killed process 2345
```

This confirmed that memory exhaustion was occurring.

---

## Step 4: Identify Memory-Hungry Processes

```bash
ps aux --sort=-%mem | head -10
```

Purpose:

- Find memory consumers
- Identify abnormal Apache growth

Example:

```text
apache2
apache2
apache2
apache2
mariadb
```

Multiple Apache processes consuming memory indicated a traffic-related issue.

---

## Step 5: Analyze Apache Access Logs

The next step was identifying what traffic was reaching the server.

## Recent Requests

```bash
tail -50 /var/log/apache2/access.log
```

---

## Check Suspicious POST Requests

```bash
grep -E "(404|500|POST)" /var/log/apache2/access.log
```

Suspicious entries appeared repeatedly:

```text
POST /xmlrpc.php
POST /xmlrpc.php
POST /xmlrpc.php
```

This immediately pointed to a WordPress XML-RPC attack.

---

## Step 6: Count XML-RPC Requests

To measure attack size:

```bash
grep -c "xmlrpc.php" /var/log/apache2/access.log
```

Result:

```text
1,108,462
```

More than one million attack requests.

---

## Step 7: Extract Attacking IP Addresses

```bash
grep "xmlrpc.php" /var/log/apache2/access.log \
| awk '{print $1}' \
| sort \
| uniq -c \
| sort -nr \
| head -20
```

Purpose:

- Identify top attackers
- Analyze attack distribution

Finding:

```text
Thousands of unique IPs
```

This confirmed a distributed botnet attack.

---

## Step 8: Search for Additional Attack Vectors

Attackers rarely stop at one technique.

## Environment File Probing

```bash
grep ".env" /var/log/apache2/access.log
```

Example:

```text
GET /.env
```

---

## WordPress Login Scanning

```bash
grep "wp-login" /var/log/apache2/access.log
```

---

## WordPress Admin Scanning

```bash
grep "wp-admin" /var/log/apache2/access.log
```

Purpose:

- Detect reconnaissance activity
- Identify additional vulnerabilities being targeted

---

## Step 9: Check Authentication Logs

```bash
grep -E "(brute|attack|hack|exploit)" /var/log/auth.log
```

Example:

```text
Invalid user exploit
```

This revealed SSH attack attempts in addition to web attacks.

---

## Step 10: Verify Network Services

```bash
netstat -tuln
```

Purpose:

- Confirm listening ports
- Detect unauthorized services

---

## Step 11: Verify Apache Configuration

```bash
apache2ctl -S
```

Purpose:

- Confirm virtual hosts
- Validate configuration

---

## Step 12: Check System Limits

```bash
ulimit -a
```

Purpose:

- Review process limits
- Verify resource constraints

---

## Step 13: Monitor in Real Time

## Monitor XML-RPC Requests

```bash
tail -f /var/log/apache2/access.log | grep xmlrpc
```

Purpose:

- Observe attack behavior live

---

## Monitor Memory Usage

```bash
watch -n 1 'free -h'
```

Purpose:

- Correlate attacks with memory spikes

---

## Investigation Findings

The investigation revealed:

- 1.1 million XML-RPC requests
- Distributed botnet activity
- Memory exhaustion
- Apache process accumulation
- MariaDB OOM kills
- Website outages

Root Cause:

```text
XML-RPC brute force / DDoS attack
```

---

## Final Resolution

The following controls were implemented:

- Apache Rate Limiting
- mod_evasive
- Fail2Ban
- XML-RPC protection
- Additional security monitoring

After implementation:

- OOM events stopped
- Apache stabilized
- Website availability improved
- Attack impact significantly reduced

---

# Conclusion

Effective incident response starts with proper investigation.

The commands covered in this guide helped identify:

- Memory exhaustion
- OOM killer activity
- XML-RPC attacks
- SSH attack attempts
- Resource bottlenecks

Every DevOps Engineer should maintain a structured investigation process and keep these commands readily available for troubleshooting production incidents.

---

**Tags:** #DevOps #Linux #WordPress #Apache #CyberSecurity #AWS #Fail2Ban #XMLRPC #IncidentResponse #SiteReliability
