# SPLUNK POWER USER  - README



### File 1: `50_queries
**The Most Comprehensive Query Reference**

**Contains:**
- 50 production-ready Splunk queries
- 5 bonus dashboard queries
- Total: 55 queries

**Organized Into 6 Sections:**

1. **User Authentication & Access Control** (8 queries)
   - Failed login detection
   - Sudo command auditing
   - Root user monitoring
   - Privilege escalation detection

2. **System Errors & Critical Failures** (10 queries)
   - Kernel panic detection
   - Out of memory alerts
   - Service crashes
   - Application segmentation faults

3. **Performance & Resource Monitoring** (9 queries)
   - CPU usage detection
   - Memory consumption alerts
   - Disk space monitoring
   - Load average tracking

4. **Network Monitoring & Connectivity** (8 queries)
   - Connection refused errors
   - Port scanning detection
   - DNS resolution failures
   - Firewall blocks

5. **Filesystem & Storage Monitoring** (7 queries)
   - File permission changes
   - File deletion auditing
   - Mount/unmount events
   - Inode exhaustion

6. **Security & Compliance Auditing** (8 queries)
   - Privilege escalation attempts
   - Firewall rule changes
   - Audit log tampering
   - Security threat detection

**Each Query Includes:**
- Purpose: Why use this query
- When to use: Practical guidance
- Copy-paste ready code



---

### File 2: `splunk_Critical/Urgent.md`
**The Daily  Quick Reference**

**Contains:**
- 55+ quick copy-paste queries
- Minimal explanation
- Maximum usability

**Organized Into 9 Quick-Reference Sections:**

1. **Critical/Urgent Queries** (5 queries)
   - For immediate crisis response
   - Errors, failed logins, disk full, memory, crashes

2. **System Health** (5 queries)
   - Error trends, reboots, CPU, memory

3. **User & Access** (8 queries)
   - Login tracking, sudo auditing, SSH keys

4. **Services & Processes** (7 queries)
   - Service restarts, crashes, systemd, cron

5. **Network** (7 queries)
   - Connections, DNS, firewall, SSH

6. **Storage & Files** (7 queries)
   - Disk errors, permissions, files, mount points

7. **Security** (6 queries)
   - Privilege escalation, unauthorized access, firewalls

8. **Performance & Load** (5 queries)
   - Load average, memory pressure, limits

9. **Dashboard Queries** (5 queries)
   - For creating dashboards



**Best For:** Day-to-day usage, quick lookups, rapid copy-paste

---

### File 3: `splunk_for_devops_english.md`
**Integration with Your DevOps Stack**

**Contains:**
- Queries for all your tools
- Setup guides for each platform
- Combined monitoring dashboards

**10 Major Sections:**

1. **Kubernetes Cluster Monitoring** (7 queries)
   - Pod health checks
   - Crash detection
   - Memory usage alerts
   - Restart spike alerts
   - Pending pods
   - Node resource pressure

2. **OpenShift Container Platform** (8 queries)
   - Failed deployments
   - Node status
   - Image pull failures
   - Operator health
   - PVC issues
   - Route traffic
   - Etcd cluster health

3. **AWS / ROSA Monitoring** (9 queries)
   - IAM permission changes
   - EC2 instance state changes
   - ROSA cluster API calls
   - Failed API calls
   - S3 bucket access
   - Root account activity
   - VPC flow logs

4. **Jenkins CI/CD Monitoring** (7 queries)
   - Build success rate
   - Failed builds alert
   - Long running builds
   - Pipeline stage failures
   - Build timeouts
   - Agent offline alerts
   - Test failures trend

5. **Docker Container Monitoring** (4 queries)
   - Container exits
   - Build failures
   - Network issues
   - Volume issues

6. **Terraform & IaC** (4 queries)
   - Plan failures
   - Apply failures
   - Infrastructure changes audit
   - Resource creation tracking

7. **Helm Chart Deployments** (3 queries)
   - Release failures
   - Release status
   - Rollback tracking

8. **ArgoCD Deployment Tracking** (3 queries)
   - Sync failures
   - Out of sync apps
   - Deployment errors

9. **Combined DevOps Dashboards** (3 queries)
   - Infrastructure health summary
   - All failed deployments
   - All alerts across stack

10. **Performance Dashboards** (3 queries)
    - Load trends
    - Success rate
    - Resource consumption

**Additional Content:**
- Step-by-step setup guide
- Useful commands for each tool
- Weekly roadmap for implementation
- DevOps-specific tips

**Best For:** Implementing Splunk across your entire infrastructure

---

##  How to Use These Files

### Quick Start (Today - 15 minutes)

**Step 1:** Open `splunk_Query.md`

**Step 2:** Find Query #1 (Show All Errors Last 24 Hours)
```spl
index=main sourcetype=syslog (ERROR OR CRITICAL) earliest=-24h | stats count by host
```

**Step 3:** Log into your Splunk instance

**Step 4:** Click Search & Reporting

**Step 5:** Paste the query in the search bar

**Step 6:** Press Enter

**Step 7:** See your results!

---

### Week 1: Learning Phase
- **Use:** `50_queries.md`
- **Activity:** Read and understand 10 queries
- **Goal:** Learn SPL syntax and logic
- **Outcome:** Understand how queries work

---

### Week 2: Practice Phase
- **Use:** `splunk_quick_RF.md`
- **Activity:** Try 15-20 different queries
- **Goal:** Get comfortable with copy-paste
- **Outcome:** Start creating your own queries

---

### Week 3: Integration Phase
- **Use:** `splunk_for_Kubernetes.md`
- **Activity:** Set up Splunk connections to your tools
- **Goal:** Get your Kubernetes, Jenkins, AWS data into Splunk
- **Outcome:** See real infrastructure data

---

### Week 4+: Automation Phase
- **Use:** All 3 files as reference
- **Activity:** Create dashboards, set up alerts
- **Goal:** Automate monitoring
- **Outcome:** Production-ready dashboards

---



## Top 10 Most Useful Queries

These are the queries you'll use most frequently:

### From Quick Reference:
1. **Show All Errors** - Daily check
2. **Failed Login Attempts** - Security
3. **Disk Almost Full** - Storage
4. **High CPU/Memory** - Performance
5. **Service Crashed** - Incident response

### From 50 Best:
6. **Kubernetes Pod Health** - DevOps
7. **Jenkins Build Success Rate** - CI/CD
8. **Failed AWS API Calls** - Cloud
9. **Privilege Escalation Attempts** - Security
10. **Infrastructure Load Trend** - Dashboard

---

