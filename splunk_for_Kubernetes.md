# SPLUNK FOR DEVOPS ENGINEERS
## Integrate with Your Infrastructure

Based on your 7 years of IT experience with Linux, Kubernetes, OpenShift, AWS, ROSA, Terraform, Jenkins, ArgoCD, Helm, and Docker.

---

## 1. KUBERNETES CLUSTER MONITORING

### Setup Splunk Connect for Kubernetes
```bash
# Add Splunk Helm repository
helm repo add splunk https://splunk.github.io/splunk-connect-for-kubernetes/

# Install Splunk Connect
helm install splunk-connect splunk/splunk-connect-for-kubernetes \
  --set splunk.hec.host=your-splunk-server \
  --set splunk.hec.port=8088 \
  --set splunk.hec.token=your-hec-token \
  -n splunk-connect --create-namespace
```

### Kubernetes Pod Health Check
```spl
index=kubernetes source=pods pod_status!=Running earliest=-1h
| stats count as failed_pods by namespace, pod_status, pod_name
| where failed_pods > 0
| alert
```

### Pod Crash Detection
```spl
index=kubernetes source=pods container_restarts>0 earliest=-24h
| stats latest(container_restarts) as restart_count by pod_name, namespace, container_name
| where restart_count > 3
| alert
```

### High Memory Usage (Kubernetes)
```spl
index=kubernetes source=pods memory_usage_bytes=* earliest=-10m
| eval memory_gb = memory_usage_bytes/1024/1024/1024
| stats avg(memory_gb) as avg_memory, max(memory_gb) as max_memory by pod_name, namespace
| where avg_memory > 2 or max_memory > 3
| alert
```

### Pod Restart Spike
```spl
index=kubernetes source=events involvedObject.kind=Pod reason=CrashLoopBackOff earliest=-1h
| stats count by involvedObject.name, involvedObject.namespace
| where count > 2
```

### Pending Pods Alert
```spl
index=kubernetes source=pods pod_status=Pending earliest=-10m
| stats count as pending_count by namespace
| where pending_count > 0
| alert
```

### Node Resource Pressure
```spl
index=kubernetes source=events reason IN ("MemoryPressure", "DiskPressure", "PIDPressure") earliest=-24h
| stats count by node_name, reason
| alert
```

---

## 2. OPENSHIFT CONTAINER PLATFORM

### Setup Splunk for OpenShift
```bash
# Install Splunk Operator for OpenShift
oc apply -f https://github.com/splunk/splunk-operator/releases/download/v2.4.0/splunk-operator-openshift.yaml

# Create standalone instance
cat <<EOF | oc apply -f -
apiVersion: enterprise.splunk.com/v4
kind: Standalone
metadata:
  name: splunk
spec:
  image: splunk/splunk:9.0
  etcVolumeStorageConfig:
    size: 10Gi
  varVolumeStorageConfig:
    size: 100Gi
EOF
```

### OpenShift Failed Deployments
```spl
index=openshift source=events type=Warning reason=FailedScheduling earliest=-24h
| stats count by involvedObject.name, reason
| alert
```

### OpenShift Node Status
```spl
index=openshift source=events (NodeNotReady OR NotReady) earliest=-24h
| stats count by node_name
| where count > 0
| alert
```

### Container Image Pull Failures
```spl
index=openshift source=events reason=Failed message="*pull*" earliest=-24h
| table _time, namespace, pod, message
| alert
```

### Operator Degraded Status
```spl
index=openshift source=cluster_operators condition!=Available earliest=-1h
| table _time, name, version, message
| alert
```

### PersistentVolumeClaim Issues
```spl
index=openshift source=events involvedObject.kind=PersistentVolumeClaim earliest=-24h
| stats count by involvedObject.name, involvedObject.namespace, reason
```

### Route Traffic Issues
```spl
index=openshift source=events involvedObject.kind=Route earliest=-24h
| regex reason="(Failed|Error)"
| table _time, involvedObject.name, reason, message
```

### Etcd Cluster Health
```spl
index=openshift component=etcd level=warn OR level=error earliest=-1h
| stats count as error_count by node
| alert
```

---

## 3. AWS / ROSA MONITORING

### Setup AWS CloudTrail Logging
```bash
# Splunk already has CloudTrail data input
# Configure in: Settings → Data Inputs → AWS CloudTrail
```

### Track IAM Permission Changes
```spl
index=aws_cloudtrail eventName IN ("PutUserPolicy", "PutGroupPolicy", "PutRolePolicy", "AttachUserPolicy")
| stats count by userIdentity.userName, eventName
| where count > 5
```

### EC2 Instance State Changes
```spl
index=aws_cloudtrail eventName IN ("StartInstances", "StopInstances", "TerminateInstances") earliest=-24h
| stats count by eventName, sourceIPAddress, awsRegion
| timechart count by eventName span=1h
```

### ROSA Cluster API Calls
```spl
index=aws_cloudtrail userAgent="rosa*" earliest=-24h
| stats count by eventName, awsRegion, userIdentity.principalId
| where count > 10
```

### Failed AWS API Calls
```spl
index=aws_cloudtrail errorCode=* errorCode!="AuthFailure" earliest=-10m
| stats count as error_count by errorCode, eventName, awsRegion
| where error_count > 5
| alert
```

### S3 Bucket Access Monitoring
```spl
index=aws_cloudtrail eventName=GetObject eventSource=s3.amazonaws.com
| stats count as access_count by requestParameters.bucketName, userIdentity.principalId
| where access_count > 100
```

### Unauthorized AWS API Attempts
```spl
index=aws_cloudtrail errorCode=AccessDenied earliest=-24h
| stats count by userIdentity.userName, eventName, sourceIPAddress
| where count > 10
```

### AWS Root Account Activity
```spl
index=aws_cloudtrail userIdentity.type=Root earliest=-24h
| table _time, eventName, sourceIPAddress, requestParameters
| alert
```

### VPC Flow Log Analysis
```spl
index=aws_vpc_flow_logs action=REJECT earliest=-1h
| stats count as rejected_connections by srcaddr, dstaddr, dstport
| where rejected_connections > 10
```

---

## 4. JENKINS CI/CD MONITORING

### Jenkins Build Success Rate
```spl
index=jenkins earliest=-7d
| stats count as total, count(eval(build_status="SUCCESS")) as success, count(eval(build_status="FAILURE")) as failure by job_name
| eval success_rate = round((success/total)*100, 2)
| where success_rate < 80
```

### Failed Builds Alert
```spl
index=jenkins build_status=FAILURE earliest=-1h
| stats count as failed_count by job_name
| where failed_count > 0
| alert
```

### Long Running Builds
```spl
index=jenkins build_duration_seconds=* earliest=-24h
| where build_duration_seconds > 3600
| table _time, job_name, build_number, build_duration_seconds
| alert
```

### Pipeline Stage Failures
```spl
index=jenkins stage_name=* stage_status=FAILED earliest=-24h
| stats count by job_name, stage_name
| alert
```

### Build Timeout Errors
```spl
index=jenkins (timeout OR "timed out" OR "Build timed out") earliest=-24h
| table _time, job_name, build_number, error_message
| stats count by job_name
```

### Jenkins Agent/Slave Offline
```spl
index=jenkins (offline OR "disconnected" OR "slave offline") earliest=-1h
| table _time, agent_name, status
| alert
```

### Test Failures Trend
```spl
index=jenkins test_failures=* earliest=-7d
| timechart avg(test_failures) by job_name span=1d
```

---

## 5. DOCKER CONTAINER MONITORING

### Docker Container Exits
```spl
index=docker (container_exit_code=* OR "container exited") earliest=-24h
| where container_exit_code != 0
| table _time, container_name, container_exit_code, reason
```

### Docker Image Build Failures
```spl
index=docker ("build failed" OR "Build error" OR ERROR) earliest=-24h
| table _time, image_name, error_message
| stats count by image_name
```

### Docker Network Issues
```spl
index=docker (network OR "network error" OR "connection refused") earliest=-24h
| stats count by container_name, error_type
```

### Docker Volume Issues
```spl
index=docker (volume OR "mount" OR "disk full") earliest=-24h
| table _time, container_name, volume_name, error_message
```

---

## 6. TERRAFORM & INFRASTRUCTURE AS CODE

### Terraform Plan Failures
```spl
index=terraform source=terraform_plan (error OR ERROR or fail) earliest=-24h
| table _time, resource_type, resource_name, error_message
| alert
```

### Terraform Apply Failures
```spl
index=terraform source=terraform_apply status=FAILURE earliest=-24h
| stats count by resource_type, error_message
| alert
```

### Infrastructure Changes Audit
```spl
index=terraform (apply OR destroy OR modify) earliest=-7d
| stats count by action, resource_type, workspace
| table resource_type, action, workspace, count
```

### Resource Creation Time Tracking
```spl
index=terraform action=apply earliest=-7d
| stats avg(duration_seconds) as avg_time, max(duration_seconds) as max_time by resource_type
```

---

## 7. HELM CHART DEPLOYMENTS

### Helm Release Failures
```spl
index=helm ("release failed" OR "install failed" OR ERROR) earliest=-24h
| table _time, release_name, namespace, chart, error_message
| alert
```

### Helm Release Status
```spl
index=helm source=releases status!=deployed earliest=-1h
| stats count by release_name, namespace, status
| where status != "deployed"
| alert
```

### Helm Upgrade Rollbacks
```spl
index=helm action=rollback earliest=-7d
| table _time, release_name, namespace, from_revision, to_revision
| stats count by release_name
```

---

## 8. ARGOCD DEPLOYMENT TRACKING

### ArgoCD Application Sync Failures
```spl
index=argocd (sync_status=Failed OR health_status=Degraded) earliest=-24h
| table _time, application_name, namespace, sync_status, health_status
| alert
```

### ArgoCD Out of Sync Applications
```spl
index=argocd sync_status=OutOfSync earliest=-1h
| stats count by application_name, namespace
| where count > 0
| alert
```

### ArgoCD Deployment Errors
```spl
index=argocd (error OR ERROR or failed) earliest=-24h
| table _time, application_name, error_message
| stats count by application_name
```

---

## 9. COMBINED DEVOPS DASHBOARD QUERY

### Daily Infrastructure Health Summary
```spl
(index=kubernetes sourcetype=pods OR index=docker OR index=jenkins OR index=terraform)
| stats count as total_events by index
| eval status = case(index="kubernetes", "K8s", index="docker", "Docker", index="jenkins", "CI/CD", index="terraform", "Infrastructure", 1=1, "Other")
| table status, total_events
```

### All Failed Deployments (Last 24 Hours)
```spl
(
  (index=kubernetes pod_status=Failed)
  OR (index=openshift reason=Failed)
  OR (index=jenkins build_status=FAILURE)
  OR (index=docker container_exit_code!=0)
  OR (index=terraform status=FAILURE)
  OR (index=helm status!=deployed)
  OR (index=argocd sync_status=Failed)
)
earliest=-24h
| stats count as failed_count by index
| sort - failed_count
```

### All Alerts Across Stack
```spl
index IN (kubernetes, openshift, jenkins, docker, terraform, helm, argocd)
earliest=-1h
| regex _raw="alert|ALERT|failed|FAILED|error|ERROR|critical|CRITICAL"
| stats count as alert_count by index
| where alert_count > 0
```

---

## 10. PERFORMANCE DASHBOARDS

### Infrastructure Load Trend
```spl
index=kubernetes earliest=-7d
| timechart 
  count as pod_events,
  count(eval(pod_status="Failed")) as failed_pods,
  count(eval(pod_status="Pending")) as pending_pods
  span=1d
```

### Deployment Success Rate (All Tools)
```spl
(index=kubernetes pod_status=Running) OR (index=jenkins build_status=SUCCESS) OR (index=helm status=deployed) OR (index=argocd health_status=Healthy)
earliest=-7d
| stats 
  count(eval(index="kubernetes" AND pod_status="Running")) as k8s_running,
  count(eval(index="jenkins" AND build_status="SUCCESS")) as jenkins_success,
  count(eval(index="helm" AND status="deployed")) as helm_deployed,
  count(eval(index="argocd" AND health_status="Healthy")) as argocd_healthy
| eval total_success = k8s_running + jenkins_success + helm_deployed + argocd_healthy
```

### Resource Consumption Trend
```spl
(index=kubernetes memory_usage_bytes=*) OR (index=docker memory_bytes=*)
earliest=-7d
| timechart avg(memory_bytes) span=1d
```

---

## SETUP GUIDE (STEP BY STEP)

### Step 1: Install Splunk Connect Components
```bash
# Kubernetes
helm install splunk-connect splunk/splunk-connect-for-kubernetes -n splunk-connect --create-namespace

# OpenShift
oc apply -f splunk-operator-openshift.yaml

# Docker (via Splunk Universal Forwarder)
docker run -d --name splunk-forwarder \
  -v /var/log:/var/log:ro \
  -e SPLUNK_START_ARGS='--accept-license' \
  -e SPLUNK_PASSWORD='your-password' \
  splunk/splunk:latest
```

### Step 2: Add Data Inputs
1. Go to Splunk Web
2. Settings → Data Inputs
3. Add: CloudTrail, VPC Flow Logs, Jenkins logs, etc.

### Step 3: Create Dashboards
1. Create searches for each tool
2. Save as dashboard panels
3. Combine into single dashboard

### Step 4: Set Up Alerts
1. Save critical searches as alerts
2. Configure notifications (Email, Slack, PagerDuty)
3. Set escalation policies

### Step 5: Monitor and Optimize
1. Check search performance
2. Optimize queries
3. Fine-tune alert thresholds

---

## USEFUL COMMANDS

### Check Kubernetes logs in Splunk:
```spl
index=kubernetes earliest=-1h
| stats count by pod_name, namespace
```

### Check Docker container status:
```spl
index=docker container_status IN (exited, running)
| stats count by container_status
```

### Check Jenkins build health:
```spl
index=jenkins 
| timechart count(eval(build_status="SUCCESS")) as success, count(eval(build_status="FAILURE")) as failure
```

### Check AWS resource changes:
```spl
index=aws_cloudtrail eventSource=* earliest=-7d
| stats count by eventName, eventSource
```

---

## NEXT STEPS FOR POWER USER

1. **Week 1:** Set up Splunk connections for Kubernetes and OpenShift
2. **Week 2:** Create dashboards for Jenkins and Docker
3. **Week 3:** Add AWS CloudTrail and configure alerts
4. **Week 4:** Create combined infrastructure dashboard
5. **Week 5+:** Automate daily/weekly reports

---


---

**Now you can monitor your entire DevOps infrastructure from one place!
