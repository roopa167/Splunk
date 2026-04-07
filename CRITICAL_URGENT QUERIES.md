# SPLUNK QUICK REFERENCE - ENGLISH VERSION
## Copy & Paste Ready Queries for Daily Use

---

##  CRITICAL/URGENT QUERIES

### 1. Show All Errors Last 24 Hours
```spl
index=main sourcetype=syslog (ERROR OR CRITICAL) earliest=-24h | stats count by host
```
**Use this when:** You need quick overview of errors

---

### 2. Failed Login Attempts - Security Alert
```spl
index=main sourcetype=syslog "Failed password" earliest=-24h | stats count by host, user | where count > 5
```
**Use this when:** You suspect brute force attack

---

### 3. Disk Almost Full - Storage Alert
```spl
index=main sourcetype=syslog df | regex _raw="([8-9][0-9]|100)%" | table host, _raw
```
**Use this when:** You need to check disk space

---

### 4. Out of Memory - Performance Alert
```spl
index=main sourcetype=syslog "Out of memory" OR "OOM killer" | stats count by host
```
**Use this when:** System is slow due to memory

---

### 5. Service Crashed - Service Alert
```spl
index=main sourcetype=syslog (crash OR "Core" OR "Segmentation fault") | table _time, host, process
```
**Use this when:** Application or service went down

---

##  SYSTEM HEALTH

### 6. Top 10 Hosts with Most Errors
```spl
index=main sourcetype=syslog ERROR | stats count by host | top 10
```
**View:** Which servers have the most problems

---

### 7. Error Trend Over Last 7 Days
```spl
index=main sourcetype=syslog ERROR earliest=-7d | timechart count by host span=1d
```
**View:** How errors are trending

---

### 8. System Reboot Events
```spl
index=main sourcetype=syslog (reboot OR "system reboot" OR "kernel boot") earliest=-7d | stats count by host
```
**View:** How many times each server rebooted

---

### 9. High CPU Usage Events
```spl
index=main sourcetype=syslog cpu_usage=* | where cpu_usage > 80 | table host, cpu_usage, _time
```
**View:** When CPU exceeded 80%

---

### 10. Memory Usage Trend
```spl
index=main sourcetype=syslog memory_usage=* | timechart avg(memory_usage) by host span=1h
```
**View:** Memory usage over time

---

##  USER & ACCESS

### 11. Who Logged In Last 24 Hours?
```spl
index=main sourcetype=syslog "Accepted" (password OR publickey) earliest=-24h | table _time, host, user
```
**View:** All successful logins

---

### 12. Failed Login Attempts - Top Users
```spl
index=main sourcetype=syslog "Failed password" earliest=-24h | top 20 user
```
**View:** Who tried to log in and failed

---

### 13. Sudo Commands Executed
```spl
index=main sourcetype=syslog sudo earliest=-24h | table _time, host, user, _raw | head 50
```
**View:** Who ran privileged commands

---

### 14. Users with Most Sudo Attempts
```spl
index=main sourcetype=syslog sudo | top 20 user
```
**View:** Who uses sudo most often

---

### 15. New User Accounts Created
```spl
index=main sourcetype=syslog (useradd OR "NEW USER") earliest=-7d | table _time, host, user
```
**View:** New accounts added to system

---

### 16. Root Access Events
```spl
index=main sourcetype=syslog user=root | stats count by host
```
**View:** How many root access events

---

### 17. Invalid Login Attempts
```spl
index=main sourcetype=syslog "Invalid user" earliest=-24h | top 20 user
```
**View:** Attempted logins with wrong usernames

---

### 18. SSH Key Changes
```spl
index=main sourcetype=syslog ("ssh-keygen" OR "authorized_keys") earliest=-7d | table _time, host, user
```
**View:** When SSH keys were added/modified

---

## 🔧 SERVICES & PROCESSES

### 19. Service Restarts in Last 24 Hours
```spl
index=main sourcetype=syslog (Started OR Stopped OR Restarted) earliest=-24h | stats count by host
```
**View:** Service restart activity

---

### 20. Failed Service Starts
```spl
index=main sourcetype=syslog (Failed OR failed) (start OR Start) earliest=-24h | table _time, host, service
```
**View:** Services that failed to start

---

### 21. Process Killed/Terminated
```spl
index=main sourcetype=syslog (SIGKILL OR Terminated OR Killed) earliest=-24h | table _time, host, process, pid
```
**View:** Processes that were forcefully terminated

---

### 22. Systemd Service Failures
```spl
index=main sourcetype=syslog systemd | regex _raw="(Failed|failed)" | table _time, host, unit, result
```
**View:** Services managed by systemd that failed

---

### 23. Cron Job Execution
```spl
index=main sourcetype=syslog CRON earliest=-24h | stats count by host
```
**View:** Cron job activity

---

### 24. Application Crashes
```spl
index=main sourcetype=syslog (Segmentation OR segfault OR "core dump") earliest=-24h | table _time, host, process
```
**View:** Applications that crashed

---

### 25. Services Listening on Ports
```spl
index=main sourcetype=syslog (LISTENING OR "listening on") | table host, service, port
```
**View:** What services are listening

---

##  NETWORK

### 26. Connection Refused Errors
```spl
index=main sourcetype=syslog "Connection refused" | stats count by host, dest_ip, dest_port | sort - count
```
**View:** Services that can't be reached

---

### 27. Firewall Blocks
```spl
index=main sourcetype=syslog (DROP OR REJECT OR blocked) | stats count by host, source_ip
```
**View:** Blocked connections

---

### 28. DNS Failures
```spl
index=main sourcetype=syslog ("DNS" OR "resolve" OR "NXDOMAIN" OR "name not found") earliest=-24h | table _time, host, domain
```
**View:** DNS resolution problems

---

### 29. Network Interface Errors
```spl
index=main sourcetype=syslog (eth OR eno OR wlan) errors | table _time, host, interface, errors
```
**View:** Network card errors

---

### 30. Timeout Errors
```spl
index=main sourcetype=syslog (timeout OR "timed out") | stats count by host, service
```
**View:** Connection timeouts

---

### 31. SSH Connections - Summary
```spl
index=main sourcetype=syslog sshd earliest=-24h | stats count by host
```
**View:** SSH activity summary

---

### 32. Port Scanning Detected
```spl
index=main sourcetype=syslog ("port scan" OR "nmap" OR "syn flood") earliest=-24h | alert
```
**Use this when:** You need security alert

---

##  STORAGE & FILES

### 33. Disk I/O Errors
```spl
index=main sourcetype=syslog ("I/O error" OR "disk error") earliest=-24h | table _time, host, device
```
**View:** Disk errors

---

### 34. File Permission Changed
```spl
index=main sourcetype=syslog chmod earliest=-7d | table _time, host, user, file
```
**View:** Permission modifications

---

### 35. File Ownership Changed
```spl
index=main sourcetype=syslog chown earliest=-7d | table _time, host, user, file
```
**View:** Ownership changes

---

### 36. File Deleted
```spl
index=main sourcetype=syslog (unlink OR "removed file") earliest=-7d | table _time, host, user, file
```
**View:** File deletions

---

### 37. Mount/Unmount Events
```spl
index=main sourcetype=syslog (mount OR unmount) earliest=-7d | table _time, host, device, mount_point
```
**View:** Storage mounting activity

---

### 38. Inode Problems
```spl
index=main sourcetype=syslog ("inode" OR "no space left") earliest=-24h | table _time, host, filesystem
```
**View:** Inode exhaustion

---

### 39. Disk 100% Full
```spl
index=main sourcetype=syslog df | regex _raw="100%" | alert
```
**Alert:** When disk is completely full

---

##  SECURITY

### 40. Privilege Escalation Attempts
```spl
index=main sourcetype=syslog (sudo OR su) (denied OR "sorry") earliest=-24h | alert
```
**Alert:** Sudo failures (possible attack)

---

### 41. Unauthorized Access Attempts
```spl
index=main sourcetype=syslog (denied OR unauthorized OR "not permitted") | stats count by host, user
```
**View:** Access denial events

---

### 42. Firewall Rules Changed
```spl
index=main sourcetype=syslog iptables (add OR delete) earliest=-7d | table _time, host, user, rule
```
**View:** Firewall modifications

---

### 43. Audit Log Tampering
```spl
index=main sourcetype=syslog auditd | stats count by host | where count > 5000
```
**View:** Excessive audit activity

---

### 44. SetUID Bit Changes
```spl
index=main sourcetype=syslog setuid earliest=-7d | table _time, host, file, action
```
**View:** Privilege escalation attempts

---

### 45. Suspicious Process Execution
```spl
index=main sourcetype=syslog (bash OR sh OR exec) fork earliest=-24h | stats count by host, process
```
**View:** Process spawning activity

---

## 📈 PERFORMANCE & LOAD

### 46. High Load Average
```spl
index=main sourcetype=syslog "load average" earliest=-1h | where load1 > 4 | table _time, host
```
**View:** System overload

---

### 47. Process Hitting Limits
```spl
index=main sourcetype=syslog ("Too many open files" OR "ulimit") earliest=-24h | table host, process
```
**View:** Resource limit errors

---

### 48. Context Switch Storms
```spl
index=main sourcetype=syslog "context switches" earliest=-1h | table _time, host
```
**View:** System stress indicators

---

### 49. High Swap Usage
```spl
index=main sourcetype=syslog swap | regex _raw="([7-9][0-9]|100)%" | table host
```
**View:** Memory pressure

---

### 50. File Descriptor Limits
```spl
index=main sourcetype=syslog "file descriptor" OR "Too many open" earliest=-24h | table host, process, pid
```
**View:** Open file limit issues

---

---

## DASHBOARD QUERIES

### 51. Daily Security Report
```spl
index=main sourcetype=syslog earliest=-24h@h latest=now
| stats count(eval(match(_raw, "Failed password"))) as failed_logins, 
  count(eval(match(_raw, "sudo"))) as sudo_commands, 
  count(eval(match(_raw, "ERROR"))) as errors 
  by host
```

### 52. Hourly Error Trend
```spl
index=main sourcetype=syslog ERROR earliest=-7d | timechart count span=1h
```

### 53. User Activity Summary
```spl
index=main sourcetype=syslog (login OR sudo) earliest=-24h | stats count by user, host | sort - count
```

### 54. Service Health Status
```spl
index=main sourcetype=syslog (Started OR Failed) earliest=-24h 
| stats count(eval(match(_raw, "Started"))) as started, count(eval(match(_raw, "Failed"))) as failed by service
```

### 55. Security Incidents Today
```spl
index=main sourcetype=syslog (sudo denied OR "Failed password" OR "Invalid user") earliest=-24h
| stats count as incidents by host | where incidents > 0
```

---

## 📌 QUICK TIPS

 Add `earliest=-24h` to make searches faster
 Use `head 100` when testing queries
 Add `| stats count by host` to group results
 Use `| where count > X` to filter
 Use `| sort - count` to sort by count descending
 Use `| table field1, field2` to show specific fields
 Use `| rename` to rename columns
 Use `| eval` to create new fields
 Use `| dedup` to remove duplicates
 Use `| alert` to convert to alert

---

## COMMON FIELD NAMES

| Field | Meaning |
|-------|---------|
| _time | Date and time |
| host | Server name |
| sourcetype | Log type (syslog, apache) |
| source | Log file path |
| _raw | Complete log message |
| user | Username |
| process | Service name |
| pid | Process ID |
| status | Event status |
| message | Event description |

---

## WHAT TO SEARCH FOR FIRST

1. **Errors:** `index=main ERROR`
2. **Specific host:** `index=main host=server-name`
3. **Specific time:** `index=main earliest=-1h`
4. **Specific user:** `index=main user=username`
5. **Specific process:** `index=main process=servicename`

---

## WHEN TO USE EACH QUERY

| Query | When to Use |
|-------|------------|
| 1-5 | Daily security checks |
| 6-10 | System health monitoring |
| 11-18 | User and access auditing |
| 19-25 | Service and process monitoring |
| 26-32 | Network troubleshooting |
| 33-39 | Storage monitoring |
| 40-49 | Security incident response |
| 50-55 | Dashboard and reporting |

---

## SAVE QUERIES FOR LATER

1. Run any query
2. Click **"Save As"**
3. Select **"Search"**
4. Give it a name
5. Click **"Save"**

Now you can find it under **Saved Searches**

---

## CREATE ALERTS

1. Run any query
2. Click **"Save As"** → **"Alert"**
3. Set trigger: **"When results > 0"**
4. Add action: **"Send Email"**
5. Enter your email
6. Click **"Save"**

---

## SCHEDULE REPORTS

1. Save query as search
2. Click **"Schedule"**
3. Set frequency: Daily, Weekly, etc
4. Set delivery: Email
5. Enter recipients
6. Click **"Save"**

---

## TROUBLESHOOTING

### Query returns no results?
- Check time range: `earliest=-24h`
- Check sourcetype: `sourcetype=syslog`
- Simplify query and test step by step

### Query is too slow?
- Add time range
- Use specific host
- Use field search instead of regex
- Add `head 100` for testing

### Can't find logs?
Try: `index=main | stats count by sourcetype`
This shows what logs you have

---

## NEXT STEPS

**Week 1:** Try 10 queries from this list
**Week 2:** Create your first dashboard
**Week 3:** Set up 3-5 alerts
**Week 4:** Schedule daily reports

---

**Happy Splunking! You now have 55+ powerful queries ready to use! 🚀**
