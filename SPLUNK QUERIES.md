# 50 BEST SPLUNK QUERIES FOR LINUX SYSTEM ADMINISTRATION

## SECTION 1: USER AUTHENTICATION & ACCESS CONTROL (Queries 1-8)

### Query 1: Failed SSH Login Attempts (Last 24 Hours)
```spl
index=main sourcetype=syslog "Failed password" OR "Failed publickey" OR "Invalid user" earliest=-24h
| stats count as failed_attempts by host, user
| where failed_attempts > 3
| sort - failed_attempts
```
**Purpose:** Detect brute force attacks on your Linux servers
**When to use:** Daily security monitoring

---

### Query 2: Successful SSH Logins by User
```spl
index=main sourcetype=syslog "Accepted publickey" OR "Accepted password" earliest=-24h
| stats count as login_count by host, user
| table host, user, login_count
| sort - login_count
```
**Purpose:** Audit who logged in and when
**When to use:** User activity auditing

---

### Query 3: Root User Activities (Privilege Escalation)
```spl
index=main sourcetype=syslog (user=root OR uid=0) (action=* OR event=*) earliest=-24h
| table _time, host, user, command, action
| sort - _time
```
**Purpose:** Monitor root access for security compliance
**When to use:** Security audits

---

### Query 4: Sudo Command Execution Audit
```spl
index=main sourcetype=syslog sudo earliest=-24h
| rex field=_raw "sudo:\s+(?<user>\w+)\s+:\s+(?<sudo_user>\w+)\s+:\s+(?<command>.*)"
| table _time, host, user, sudo_user, command
| stats count by user, command
```
**Purpose:** Track who ran privileged commands and what they executed
**When to use:** Compliance reporting

---

### Query 5: Failed Sudo Attempts
```spl
index=main sourcetype=syslog "sudo" "sorry" earliest=-24h
| stats count as failed_sudo_attempts by host, user
| where failed_sudo_attempts > 0
| sort - failed_sudo_attempts
```
**Purpose:** Detect privilege escalation attempts
**When to use:** Security incident investigation

---

### Query 6: User Account Creation Events
```spl
index=main sourcetype=syslog (useradd OR "useradd" OR "new user") earliest=-7d
| table _time, host, user, command
| stats count by user
```
**Purpose:** Track new user account creation
**When to use:** Compliance and audit trails

---

### Query 7: User Account Deletion Events
```spl
index=main sourcetype=syslog (userdel OR "user deleted") earliest=-7d
| table _time, host, deleted_user, admin_user
```
**Purpose:** Monitor account removal
**When to use:** User lifecycle management

---

### Query 8: Failed Login Attempts with Source IP
```spl
index=main sourcetype=syslog "Failed password" earliest=-24h
| rex field=_raw "Failed password for (?<user>\w+) from (?<source_ip>\d+\.\d+\.\d+\.\d+)"
| stats count as failed_attempts by source_ip, user
| where failed_attempts > 10
| sort - failed_attempts
```
**Purpose:** Identify source of brute force attacks
**When to use:** Network security investigation

---

## SECTION 2: SYSTEM ERRORS & CRITICAL FAILURES (Queries 9-18)

### Query 9: All Critical System Errors (Last 24 Hours)
```spl
index=main sourcetype=syslog (CRITICAL OR FATAL OR EMERG OR ALERT OR ERROR) earliest=-24h
| table _time, host, process, _raw
| stats count by host
| sort - count
```
**Purpose:** Get overview of critical system issues
**When to use:** Daily health check

---

### Query 10: Kernel Panic and System Crashes
```spl
index=main sourcetype=syslog ("kernel panic" OR "Oops" OR "BUG" OR "CPU hung" OR "watchdog timer") earliest=-7d
| stats count as panic_count by host
| where panic_count > 0
| alert
```
**Purpose:** Detect system crashes
**When to use:** Hardware/system stability monitoring

---

### Query 11: Out of Memory (OOM) Killer Events
```spl
index=main sourcetype=syslog ("Out of memory" OR "OOM killer" OR "oom-kill" OR "OOM: Killed process") earliest=-24h
| table _time, host, process, pid
| stats count by host, process
| sort - count
```
**Purpose:** Monitor memory exhaustion events
**When to use:** Memory leak investigation

---

### Query 12: Segmentation Faults (Application Crashes)
```spl
index=main sourcetype=syslog ("segmentation fault" OR "segfault" OR "SIGSEGV" OR "Segmentation violation") earliest=-24h
| stats count as segfault_count by host, process
| where segfault_count > 0
| alert
```
**Purpose:** Identify faulty or unstable applications
**When to use:** Application debugging

---

### Query 13: Service Start and Stop Events
```spl
index=main sourcetype=syslog ("Started" OR "Stopped" OR "Restarted" OR "started service") earliest=-24h
| table _time, host, service, action
| stats count by host, service, action
```
**Purpose:** Track service lifecycle changes
**When to use:** Infrastructure change tracking

---

### Query 14: Systemd Service Failures
```spl
index=main sourcetype=syslog systemd (Failed OR Error OR error) earliest=-24h
| rex field=_raw "(?<service>\S+).service"
| table _time, host, service, result, message
| stats count by service
```
**Purpose:** Monitor systemd service failures
**When to use:** Service health monitoring

---

### Query 15: Application Core Dumps
```spl
index=main sourcetype=syslog ("core dump" OR "core dumped" OR "created core" OR "fatal") earliest=-7d
| table _time, host, process, signal, pid
| stats count by host, process
```
**Purpose:** Detect application crashes and core dumps
**When to use:** Application stability monitoring

---

### Query 16: Disk I/O Errors
```spl
index=main sourcetype=syslog ("I/O error" OR "disk error" OR "read error" OR "write error" OR "EXT4-fs error") earliest=-24h
| table _time, host, device, error_message
| stats count by host, device
| alert
```
**Purpose:** Identify disk hardware issues
**When to use:** Storage health monitoring

---

### Query 17: Connection Timeout Errors
```spl
index=main sourcetype=syslog ("timeout" OR "timed out" OR "connection timeout" OR "operation timed out") earliest=-24h
| stats count as timeout_count by host, process, destination
| where timeout_count > 5
| sort - timeout_count
```
**Purpose:** Find connectivity and network issues
**When to use:** Network troubleshooting

---

### Query 18: Kernel Module Loading Failures
```spl
index=main sourcetype=syslog ("Cannot load" OR "Failed to load" OR "module" OR "modprobe") earliest=-7d
| table _time, host, module, error_message
| stats count by module
```
**Purpose:** Detect driver and kernel module problems
**When to use:** Hardware compatibility issues

---

## SECTION 3: PERFORMANCE & RESOURCE MONITORING (Queries 19-27)

### Query 19: High CPU Usage Detection
```spl
index=main sourcetype=syslog cpu (high OR usage OR "100%" OR "99%") earliest=-1h
| table _time, host, process, cpu_percentage
| where cpu_percentage > 80
| stats avg(cpu_percentage) as avg_cpu, max(cpu_percentage) as max_cpu by host
```
**Purpose:** Find CPU-intensive processes
**When to use:** Performance optimization

---

### Query 20: High Memory Consumption Alert
```spl
index=main sourcetype=syslog memory (high OR usage OR "90%" OR "95%") earliest=-1h
| table _time, host, process, memory_usage
| where memory_usage > 80
| stats latest(memory_usage) as latest_memory by host
| alert
```
**Purpose:** Detect memory leaks and exhaustion
**When to use:** Memory leak investigation

---

### Query 21: Disk Space Utilization Alert
```spl
index=main sourcetype=syslog (df OR disk_free OR disk_used) earliest=-1h
| regex _raw="(?<disk_percent>\d+)%"
| where disk_percent > 85
| table host, filesystem, disk_percent
| sort - disk_percent
```
**Purpose:** Monitor disk space before running out
**When to use:** Storage capacity planning

---

### Query 22: Inode Usage Alert
```spl
index=main sourcetype=syslog (inode OR "inodes used" OR "Inode") earliest=-24h
| regex _raw="(?<inode_percent>\d+)%"
| where inode_percent > 80
| table host, filesystem, inode_percent
```
**Purpose:** Detect inode limit issues
**When to use:** Filesystem health check

---

### Query 23: Too Many Open Files Error
```spl
index=main sourcetype=syslog ("Too many open files" OR "file descriptor" OR "ulimit") earliest=-24h
| stats count as open_files_errors by host, process
| sort - open_files_errors
```
**Purpose:** Find processes hitting file descriptor limits
**When to use:** Resource limit investigation

---

### Query 24: System Load Average Spike
```spl
index=main sourcetype=syslog "load average" earliest=-1h
| rex field=_raw "load average[s]*:\s*(?<load1>[\d.]+)\s*,\s*(?<load5>[\d.]+)\s*,\s*(?<load15>[\d.]+)"
| where load1 > 4
| table _time, host, load1, load5, load15
| alert
```
**Purpose:** Detect system overload
**When to use:** Performance troubleshooting

---

### Query 25: Process Fork/Creation Limit Reached
```spl
index=main sourcetype=syslog ("fork:" OR "process" OR "pid") (limit OR "no space") earliest=-24h
| stats count by host
```
**Purpose:** Monitor process creation limits
**When to use:** System stability checks

---

### Query 26: High Swap Usage Alert
```spl
index=main sourcetype=syslog swap (usage OR used) earliest=-1h
| regex _raw="(?<swap_percent>\d+)%"
| where swap_percent > 80
| table host, swap_percent
| alert
```
**Purpose:** Detect memory pressure and swapping
**When to use:** Performance analysis

---

### Query 27: Context Switches and Interrupts
```spl
index=main sourcetype=syslog (context OR interrupt OR "CS" OR "IN") earliest=-1h
| regex _raw="context switches:\s*(?<cs_count>\d+)"
| table _time, host, cs_count
```
**Purpose:** Monitor system interrupts and context switches
**When to use:** System performance analysis

---

## SECTION 4: NETWORK MONITORING & CONNECTIVITY (Queries 28-35)

### Query 28: Connection Refused Errors
```spl
index=main sourcetype=syslog ("Connection refused" OR "ECONNREFUSED" OR "connection refused") earliest=-24h
| stats count as refused_count by host, destination_ip, destination_port
| where refused_count > 5
| sort - refused_count
| alert
```
**Purpose:** Find unreachable services
**When to use:** Service availability troubleshooting

---

### Query 29: Port Scanning Detection (Security)
```spl
index=main sourcetype=syslog ("port scan" OR "nmap" OR "syn flood" OR "UDP scan") earliest=-24h
| stats count by host, source_ip, destination_port
| where count > 20
| alert
```
**Purpose:** Security threat detection
**When to use:** Intrusion detection

---

### Query 30: Firewall Rules Drops/Blocks
```spl
index=main sourcetype=syslog (DROP OR REJECT OR "Connection blocked" OR "DROPPED") earliest=-24h
| stats count as blocked_count by source_ip, destination_port, host
| sort - blocked_count
```
**Purpose:** Monitor firewall activity
**When to use:** Network security auditing

---

### Query 31: DNS Resolution Failures
```spl
index=main sourcetype=syslog ("Name or service not known" OR "DNS error" OR "NXDOMAIN" OR "failed to resolve") earliest=-24h
| stats count as dns_failures by host, domain
| sort - dns_failures
```
**Purpose:** Detect DNS resolution problems
**When to use:** DNS troubleshooting

---

### Query 32: Network Interface Errors
```spl
index=main sourcetype=syslog (eth OR eno OR wlan OR net) (errors OR dropped OR collisions) earliest=-24h
| regex _raw="(?<error_count>\d+) errors"
| table _time, host, interface, error_count
| stats count by host, interface
```
**Purpose:** Monitor network card and driver issues
**When to use:** Network hardware health check

---

### Query 33: TCP Connection Timeouts
```spl
index=main sourcetype=syslog ("tcp_abort_on_timeout" OR "connection_closed" OR "timed out" OR "TCP") earliest=-24h
| stats count as timeout_count by host, destination_ip
| where timeout_count > 5
| sort - timeout_count
```
**Purpose:** Find slow or hanging connections
**When to use:** Network performance troubleshooting

---

### Query 34: SSH Connection Summary
```spl
index=main sourcetype=syslog sshd earliest=-24h
| regex _raw="(Accepted|Failed|Invalid|Disconnected)"
| stats count by host, _raw
| sort - count
```
**Purpose:** SSH activity overview
**When to use:** Access monitoring

---

### Query 35: Listening Ports and Services
```spl
index=main sourcetype=syslog (LISTENING OR "listening on" OR "bind") earliest=-24h
| rex field=_raw "(?<service>\w+).*(?<port>\d+)"
| table host, service, port
| stats count by service
```
**Purpose:** Monitor exposed services and listening ports
**When to use:** Network security audit

---

## SECTION 5: FILESYSTEM AND STORAGE MONITORING (Queries 36-42)

### Query 36: File Permission Changes (Security Audit)
```spl
index=main sourcetype=syslog (chmod OR "permission denied" OR "mode change") earliest=-7d
| table _time, host, user, file, old_perms, new_perms
| stats count by user
```
**Purpose:** Security audit for permission modifications
**When to use:** Compliance reporting

---

### Query 37: File Ownership Changes
```spl
index=main sourcetype=syslog chown earliest=-7d
| table _time, host, user, file, old_owner, new_owner
| stats count by user
```
**Purpose:** Track file ownership modifications
**When to use:** Security auditing

---

### Query 38: SetUID/SetGID Bit Changes
```spl
index=main sourcetype=syslog ("setuid" OR "setgid" OR "sticky bit") earliest=-7d
| table _time, host, file, action, permission
| alert
```
**Purpose:** Detect privilege escalation attempts
**When to use:** Security threat detection

---

### Query 39: File Deletion Audit Log
```spl
index=main sourcetype=syslog (unlink OR "removed file" OR "deleted" OR "rm:") earliest=-7d
| table _time, host, user, file, path
| stats count by user
```
**Purpose:** Track file deletions
**When to use:** Data loss investigation

---

### Query 40: Mount and Unmount Events
```spl
index=main sourcetype=syslog (mount OR unmount OR "remount" OR "filesystem") earliest=-7d
| table _time, host, device, mount_point, action, user
| stats count by action
```
**Purpose:** Monitor storage changes
**When to use:** Infrastructure change tracking

---

### Query 41: Symbolic Link Creation (Security)
```spl
index=main sourcetype=syslog (symlink OR "symbolic link" OR "ln -s") earliest=-7d
| table _time, host, user, source_file, target_file
| alert
```
**Purpose:** Detect privilege escalation attempts using symlinks
**When to use:** Security incident investigation

---

### Query 42: File Integrity Changes
```spl
index=main sourcetype=syslog (CHANGED OR "changed" OR "integrity" OR "checksum" OR "modified") earliest=-7d
| stats count as integrity_changes by host, file
| alert
```
**Purpose:** Detect unauthorized file modifications
**When to use:** Intrusion detection

---

## SECTION 6: SECURITY AND COMPLIANCE AUDITING (Queries 43-50)

### Query 43: Failed Authentication Attempts Summary
```spl
index=main sourcetype=syslog (auth OR authentication) (Failed OR FAILURE OR denied OR invalid) earliest=-24h
| stats count as failed_auth_count by host, user, authentication_method
| where failed_auth_count > 5
| alert
```
**Purpose:** Alert on brute force authentication attempts
**When to use:** Security monitoring

---

### Query 44: Privilege Escalation Attempts
```spl
index=main sourcetype=syslog (sudo OR su OR "sudo -u") (denied OR DENIED OR "sorry" OR "incorrect password") earliest=-24h
| table _time, host, user, command, result
| alert
```
**Purpose:** Detect privilege escalation attacks
**When to use:** Security threat detection

---

### Query 45: Cron Job Modifications
```spl
index=main sourcetype=syslog (cron OR crond) earliest=-7d
| regex _raw="(CMD|REPLACE|DELETE)"
| table _time, host, user, command, action
| stats count by action
```
**Purpose:** Monitor scheduled task changes
**When to use:** Configuration management audit

---

### Query 46: PAM (Pluggable Authentication Module) Errors
```spl
index=main sourcetype=syslog pam (error OR Error OR ERROR) earliest=-24h
| stats count as pam_errors by host, pam_module, error_type
| alert
```
**Purpose:** Identify authentication system issues
**When to use:** Access control troubleshooting

---

### Query 47: SSH Key Management Events
```spl
index=main sourcetype=syslog (ssh-keygen OR "ssh-agent" OR "known_hosts" OR "authorized_keys") earliest=-7d
| table _time, host, user, action, key_file
```
**Purpose:** Track SSH key management activities
**When to use:** Security compliance auditing

---

### Query 48: Process Termination/Kill Signals
```spl
index=main sourcetype=syslog (SIGKILL OR SIGSEGV OR "Terminated" OR "Killed" OR "killed process") earliest=-24h
| table _time, host, process, signal, pid, reason
| stats count as kill_count by process
```
**Purpose:** Detect suspicious process terminations
**When to use:** Security incident investigation

---

### Query 49: Firewall Rule Changes
```spl
index=main sourcetype=syslog (iptables OR firewall OR "ufw" OR "ufw:") (add OR delete OR modify OR INSERT OR APPEND) earliest=-7d
| table _time, host, user, action, rule, chain
| alert
```
**Purpose:** Audit firewall configuration changes
**When to use:** Network security compliance

---

### Query 50: Audit Log Tampering Detection
```spl
index=main sourcetype=syslog (auditd OR audit OR "audit.log") earliest=-24h
| regex _raw="(exec=|key=|rule=|watch=)"
| stats count as audit_events by host
| where audit_events > 5000
| alert
```
**Purpose:** Detect log tampering or excessive audit activity
**When to use:** Security compliance and forensics

---

## BONUS QUERIES FOR DASHBOARDS

### Query 51: Daily Security Report
```spl
index=main sourcetype=syslog earliest=-24h@h latest=now
| stats 
  count as total_events,
  count(eval(match(_raw, "Failed password"))) as failed_logins,
  count(eval(match(_raw, "sudo"))) as sudo_commands,
  count(eval(match(_raw, "ERROR|CRITICAL"))) as critical_errors
  by host
| table host, total_events, failed_logins, sudo_commands, critical_errors
```

### Query 52: System Health Score
```spl
index=main sourcetype=syslog earliest=-24h@h latest=now
| stats 
  count as total_events,
  count(eval(match(_raw, "error"))) as errors,
  count(eval(match(_raw, "warning"))) as warnings
  by host
| eval health_score = round(((total_events - (errors*2) - warnings)/total_events)*100, 2)
| where health_score < 80
| alert
```

### Query 53: Sudo Compliance Report (Last 7 Days)
```spl
index=main sourcetype=syslog sudo earliest=-7d
| stats 
  count as commands_run,
  dc(user) as unique_users,
  dc(command) as unique_commands
  by host
| table host, commands_run, unique_users, unique_commands
```

### Query 54: Performance Trend (Last 7 Days)
```spl
index=main sourcetype=syslog earliest=-7d
| timechart 
  count as total_events,
  count(eval(match(_raw, "error"))) as error_count,
  count(eval(match(_raw, "warning"))) as warning_count
  span=1d
```

### Query 55: User Activity Summary (Last 24 Hours)
```spl
index=main sourcetype=syslog (login OR user OR sudo OR Accepted) earliest=-24h
| stats 
  count as total_activity,
  count(eval(match(_raw, "Failed"))) as failed_attempts
  by user, host
| where total_activity > 5
| sort - total_activity
```

---

## HOW TO USE THESE QUERIES

### Step 1: Copy the Query
Select the entire query from the code block above.

### Step 2: Open Splunk
Go to your Splunk instance (usually http://your-splunk:8000)

### Step 3: Click "Search & Reporting"

### Step 4: Paste Query
Click in the search bar and paste your query.

### Step 5: Click Search
Press Enter or click the Search button.

### Step 6: View Results
Results appear in seconds to minutes.

### Step 7: Save as Dashboard (Optional)
- Click "Save As"
- Select "Dashboard"
- Give it a name
- Click Save

---

## QUICK CUSTOMIZATION GUIDE

### Change Time Range
Replace `earliest=-24h` with:
- `earliest=-1h` (Last 1 hour)
- `earliest=-7d` (Last 7 days)
- `earliest=-30d` (Last 30 days)

### Change Host
Replace `host=*` or add `host=your_server_name`

### Change Alert Threshold
Replace `count > 5` with your desired number

### Add Email Alert
After saving as search:
1. Click "Alert"
2. Set trigger: "When results > 0"
3. Add action: "Send Email"

---

## FIELD NAMES REFERENCE

| Field | Meaning | Example |
|-------|---------|---------|
| _time | Timestamp | 2024-01-15 10:30:45 |
| host | Server hostname | web-server-01 |
| sourcetype | Log type | syslog |
| source | Log file path | /var/log/auth.log |
| _raw | Complete log message | Full unformatted log |
| user | Username | admin |
| process | Service/process name | sshd |
| pid | Process ID | 12345 |
| status | Event status | ERROR, WARNING, SUCCESS |
| message | Event description | Connection timeout |

---

## OPTIMIZATION TIPS FOR FAST SEARCHES

✅ Always add `earliest=-24h` to limit data
✅ Use specific `sourcetype` to narrow results
✅ Filter at search level (not with pipes)
✅ Add `head 100` when testing
✅ Use `where` after `stats`, not before
✅ Use `fields` to show only necessary columns
✅ Use `dedup` at the end, not beginning

---

## COMMON MISTAKES TO AVOID

❌ Searching entire index without time limit
❌ Using `| search` instead of field search
❌ Running complex `regex` on large datasets
❌ Using `where` before `stats`
❌ Forgetting to add `alert` to critical queries

---

## NEXT STEPS

1. **Week 1:** Try 5-10 queries from this list
2. **Week 2:** Create 2-3 dashboards
3. **Week 3:** Set up alerts for critical queries
4. **Week 4:** Automate daily/weekly reports

---

## NEED HELP?

### Test if Splunk is working:
```spl
index=main | head 100
```

### See what data you have:
```spl
index=main | stats count by sourcetype
```

### See recent logs:
```spl
index=main earliest=-1h | head 20
```

---

**Happy Splunking! Use these queries to become a Splunk Power User! 🚀**
