# OpenShift Troubleshooting Scenarios – High Retrieval Playbook

Quick reference for on-call incidents. Commands first, reasoning second.

---

## 1. Pod Stuck in Pending

**Trigger symptom**
- Pod shows Pending status for more than 30 seconds

**What is usually wrong**
- No node has enough resources (CPU/Memory)
- PVC not available or bound
- Node selector or affinity rules prevent scheduling
- Taints on nodes without matching tolerations
- Cluster capacity maxed out

**Immediate checks (COMMANDS FIRST)**
```bash
oc get pods
oc describe pod <pod-name>
oc get nodes
oc describe node <node-name>
oc get pvc
```

**Mental debugging flow**
- Check Events section in describe output first
- Look for "FailedScheduling" message
- Verify resource requests vs node capacity
- Check if PVC exists and is Bound
- Verify node selectors match available nodes
- Check if nodes are cordoned or have taints

**Typical fix**
- Scale down other workloads or add nodes
- Create or fix PVC
- Adjust node selector or tolerations
- Uncordon nodes if applicable

**One-line KT sentence**
"Pod is Pending because scheduler cannot find a suitable node due to resource constraints, missing PVC, or node selector mismatch."

---

## 2. Pod Stuck in ContainerCreating

**Trigger symptom**
- Pod status remains ContainerCreating for extended time

**What is usually wrong**
- ConfigMap or Secret referenced but not found
- PVC mount failing
- Image pull in progress but slow network
- CNI network setup failing
- Security context or SCC blocking container creation

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc get configmap
oc get secret
oc get pvc
oc get events --sort-by='.lastTimestamp'
```

**Mental debugging flow**
- Check Events section for mount errors
- Verify all ConfigMaps and Secrets exist
- Check PVC status and availability
- Look for "MountVolume.SetUp failed" errors
- Check node kubelet logs if no clear Events

**Typical fix**
- Create missing ConfigMap or Secret
- Fix PVC provisioning issues
- Wait for slow image pull to complete
- Adjust security context or grant SCC

**One-line KT sentence**
"Pod is stuck in ContainerCreating because volumes cannot mount, usually due to missing ConfigMap, Secret, or PVC binding issues."

---

## 3. Pod CrashLoopBackOff

**Trigger symptom**
- Pod restarts repeatedly with increasing backoff delay

**What is usually wrong**
- Application exits immediately due to config error
- Missing environment variables or config files
- Application cannot connect to dependencies
- Port already in use or permission denied
- Liveness probe killing healthy startup

**Immediate checks (COMMANDS FIRST)**
```bash
oc get pods
oc logs <pod-name>
oc logs <pod-name> --previous
oc describe pod <pod-name>
oc get events
```

**Mental debugging flow**
- Read logs from current and previous containers
- Check exit code in describe output
- Look for application-level errors in logs
- Verify environment variables are set
- Check if liveness probe timeout is too aggressive
- Test dependencies are reachable

**Typical fix**
- Fix application configuration
- Add missing environment variables
- Adjust liveness probe initialDelaySeconds
- Fix network connectivity to dependencies
- Check file permissions in container

**One-line KT sentence**
"Pod is in CrashLoopBackOff because the application exits immediately, usually due to misconfiguration, missing dependencies, or aggressive probe settings."

---

## 4. Pod Restarting Continuously

**Trigger symptom**
- Pod restart count keeps incrementing without stabilizing

**What is usually wrong**
- Application crashes intermittently
- Memory leak causing OOM over time
- Liveness probe failing intermittently
- Deadlock or unhandled exceptions
- Resource limits too low for workload

**Immediate checks (COMMANDS FIRST)**
```bash
oc get pods
oc describe pod <pod-name>
oc logs <pod-name> --previous
oc logs <pod-name> -f
oc top pod <pod-name>
```

**Mental debugging flow**
- Check restart count and last restart time
- Look at previous container logs for crash reason
- Monitor live logs for patterns before crash
- Check memory usage approaching limits
- Review liveness probe configuration
- Look for OOMKilled in termination reason

**Typical fix**
- Increase memory limits if OOM
- Fix application bugs causing crashes
- Adjust liveness probe settings
- Add proper error handling in app
- Scale horizontally if load related

**One-line KT sentence**
"Pod keeps restarting because the application crashes periodically from OOM, failed health checks, or unhandled errors under load."

---

## 5. Pod OOMKilled

**Trigger symptom**
- Pod shows OOMKilled in status or last termination reason

**What is usually wrong**
- Memory limit set too low for workload
- Memory leak in application
- Traffic spike causing memory spike
- No memory limits causing node OOM
- Inefficient code or data structures

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc get pod <pod-name> -o yaml | grep -A 5 resources
oc top pod <pod-name>
oc logs <pod-name> --previous
oc get events
```

**Mental debugging flow**
- Check Last State for OOMKilled status
- Compare memory limit vs actual usage before kill
- Review application logs for memory growth
- Check if limit equals request
- Look for memory leak patterns in logs
- Determine if legitimate usage or leak

**Typical fix**
- Increase memory limits to match actual needs
- Fix application memory leak if identified
- Add or adjust memory limits if missing
- Optimize application memory usage
- Use horizontal scaling for traffic spikes

**One-line KT sentence**
"Pod was OOMKilled because memory usage exceeded the limit, either from undersized limits, memory leaks, or traffic spikes."

---

## 6. ImagePullBackOff

**Trigger symptom**
- Pod shows ImagePullBackOff status with increasing retry delay

**What is usually wrong**
- Image does not exist in registry
- Image tag is wrong or typo in name
- No pull secret configured for private registry
- Pull secret exists but wrong credentials
- Network cannot reach registry

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc get pod <pod-name> -o yaml | grep image
oc get secret
oc describe secret <pull-secret>
oc get events
```

**Mental debugging flow**
- Check Events for exact error message
- Verify image name and tag spelling
- Confirm image exists in registry
- Check if registry is private requiring auth
- Verify pull secret linked to service account
- Test network connectivity to registry from node

**Typical fix**
- Correct image name or tag
- Create image pull secret with registry credentials
- Link secret to default or custom service account
- Fix network or firewall rules to registry
- Push image to registry if missing

**One-line KT sentence**
"ImagePullBackOff occurs because OpenShift cannot pull the image due to wrong name, missing credentials, or network issues with the registry."

---

## 7. ErrImagePull

**Trigger symptom**
- Pod shows ErrImagePull status on first pull attempt

**What is usually wrong**
- Same as ImagePullBackOff but first attempt
- Registry authentication failing
- Registry unreachable or down
- Invalid image reference format
- Rate limit hit on public registry

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc get events --sort-by='.lastTimestamp'
oc get pod <pod-name> -o jsonpath='{.spec.containers[*].image}'
oc get secret -n <namespace>
```

**Mental debugging flow**
- Read exact error from Events section
- Verify image path format is correct
- Check if registry requires authentication
- Test registry accessibility from cluster
- Verify pull secret exists and is correct
- Check for typos in image reference

**Typical fix**
- Same as ImagePullBackOff resolution
- Wait if transient registry issue
- Create or update image pull secret
- Fix image reference syntax
- Use different registry or mirror

**One-line KT sentence**
"ErrImagePull is the initial failure before ImagePullBackOff, indicating the first attempt to pull the image failed for the same reasons."

---

## 8. Deployment Not Creating Pods

**Trigger symptom**
- Deployment exists but no pods are created

**What is usually wrong**
- Deployment replicas set to 0
- Deployment paused state
- Resource quota exceeded
- Deployment selector does not match template labels
- Syntax error in deployment spec

**Immediate checks (COMMANDS FIRST)**
```bash
oc get deployment
oc describe deployment <deployment-name>
oc get replicaset
oc describe replicaset <rs-name>
oc get resourcequota
oc get events
```

**Mental debugging flow**
- Check deployment replicas count
- Verify deployment is not paused
- Look at ReplicaSet status and conditions
- Check Events for quota errors
- Verify selector matches pod template labels
- Check if ReplicaSet was created at all

**Typical fix**
- Scale deployment to desired replicas
- Resume deployment if paused
- Increase or fix resource quotas
- Fix label selector mismatch
- Correct deployment YAML syntax

**One-line KT sentence**
"Deployment not creating pods because replicas is 0, deployment is paused, quota exceeded, or selector mismatch with pod labels."

---

## 9. Deployment Not Updating After Image Change

**Trigger symptom**
- Changed image in deployment but pods still run old version

**What is usually wrong**
- Image tag is same and pull policy is IfNotPresent
- Deployment paused state
- Cached image on node with same tag
- Did not apply the change correctly
- Rollout stuck or paused

**Immediate checks (COMMANDS FIRST)**
```bash
oc get deployment <deployment-name> -o yaml | grep image
oc describe deployment <deployment-name>
oc rollout status deployment/<deployment-name>
oc rollout history deployment/<deployment-name>
oc get pods -o wide
oc describe pod <pod-name> | grep Image
```

**Mental debugging flow**
- Verify deployment spec has new image
- Check if rollout is progressing or stuck
- Look at pod image to confirm old version
- Check imagePullPolicy setting
- Verify no paused rollout state
- Check rollout history for changes

**Typical fix**
- Trigger rollout restart if using same tag
- Resume deployment if paused
- Change imagePullPolicy to Always for latest tag
- Manually rollout restart the deployment
- Use immutable tags instead of latest

**One-line KT sentence**
"Deployment not updating because image tag is unchanged with IfNotPresent policy, or rollout is paused preventing new pods."

---

## 10. Old Pods Not Terminating During Rollout

**Trigger symptom**
- New pods created but old pods remain running

**What is usually wrong**
- PreStop hook hanging or failing
- Application not handling SIGTERM gracefully
- Long terminationGracePeriodSeconds exceeded
- Pod has finalizers blocking deletion
- PDB preventing pod termination

**Immediate checks (COMMANDS FIRST)**
```bash
oc get pods
oc describe pod <old-pod-name>
oc get pod <old-pod-name> -o yaml | grep -A 5 finalizers
oc get pdb
oc describe deployment <deployment-name>
oc get events
```

**Mental debugging flow**
- Check pod status for Terminating state
- Look at finalizers on pod
- Check Events for termination issues
- Verify terminationGracePeriodSeconds
- Check PDB maxUnavailable setting
- Look for preStop hook configuration

**Typical fix**
- Delete finalizers if stuck
- Force delete pod if necessary
- Fix preStop hook if hanging
- Adjust PDB constraints
- Reduce terminationGracePeriodSeconds
- Fix application SIGTERM handling

**One-line KT sentence**
"Old pods not terminating because of stuck finalizers, long grace periods, PDB constraints, or application not handling shutdown signals."

---

## 11. Service Not Reachable Inside Cluster

**Trigger symptom**
- Cannot curl service from another pod using cluster IP or DNS

**What is usually wrong**
- Service selector does not match pod labels
- No pods backing the service (endpoints empty)
- Pods not listening on service target port
- Network policy blocking traffic
- Wrong service port or target port

**Immediate checks (COMMANDS FIRST)**
```bash
oc get svc <service-name>
oc describe svc <service-name>
oc get endpoints <service-name>
oc get pods --show-labels
oc describe pod <pod-name>
oc get networkpolicy
```

**Mental debugging flow**
- Check if service has endpoints
- Verify service selector matches pod labels
- Confirm pods are in Ready state
- Test pod directly by IP and port
- Check targetPort matches container port
- Look for network policies restricting access

**Typical fix**
- Fix service selector to match pod labels
- Fix pod readiness issues to populate endpoints
- Correct service port or targetPort
- Update or delete blocking network policy
- Verify application listening on correct port

**One-line KT sentence**
"Service not reachable because selector does not match pods, no endpoints available, or network policy is blocking the connection."

---

## 12. Service Not Reachable From Outside

**Trigger symptom**
- External clients cannot reach service via route or load balancer

**What is usually wrong**
- No route or ingress configured
- Route pointing to wrong service
- Service type ClusterIP not accessible externally
- Router or ingress controller issue
- Firewall blocking external access

**Immediate checks (COMMANDS FIRST)**
```bash
oc get route
oc describe route <route-name>
oc get svc
oc get endpoints <service-name>
oc get pods -n openshift-ingress
curl -I http://<route-hostname>
```

**Mental debugging flow**
- Verify route exists and has hostname
- Check route backend service is correct
- Confirm service has endpoints
- Test service internally first
- Check router pods are running
- Verify DNS resolves route hostname

**Typical fix**
- Create route if missing
- Change service type to LoadBalancer or create route
- Fix route service reference
- Restart router pods if unhealthy
- Fix DNS or firewall rules

**One-line KT sentence**
"Service not reachable externally because no route exists, route misconfigured, or router pods are failing."

---

## 13. Route Returns 503 / 404

**Trigger symptom**
- Route accessible but returns HTTP 503 or 404 error

**What is usually wrong**
- No healthy endpoints behind service (503)
- Application not listening on correct path (404)
- Service name wrong in route (503)
- Pods failing readiness checks (503)
- Router cannot reach service endpoints

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe route <route-name>
oc get endpoints <service-name>
oc get pods
oc describe pod <pod-name>
oc logs <pod-name>
curl -v http://<route-hostname>
```

**Mental debugging flow**
- Check service has active endpoints
- Verify pods are in Ready state
- Check route service name is correct
- Test service cluster IP directly from pod
- Check application logs for errors
- Verify application path matches request

**Typical fix**
- Fix pod readiness probe issues
- Correct route service backend reference
- Fix application to listen on correct port
- Scale up service if no pods running
- Check application routing logic

**One-line KT sentence**
"Route returns 503 when service has no healthy endpoints, or 404 when application does not serve the requested path."

---

## 14. Route Works Intermittently

**Trigger symptom**
- Route accessible sometimes but fails other times

**What is usually wrong**
- Some pods failing health checks intermittently
- One or more pods in unhealthy state
- Network issues between router and pods
- Application crashes or hangs randomly
- Session affinity causing routing to bad pod

**Immediate checks (COMMANDS FIRST)**
```bash
oc get pods
oc get endpoints <service-name>
oc logs <pod-name>
oc describe pod <pod-name>
oc get route <route-name> -o yaml | grep -i sticky
watch oc get endpoints <service-name>
```

**Mental debugging flow**
- Check endpoint list for changes
- Verify all pods are consistently Ready
- Test each pod individually by IP
- Check for failing health probes
- Look for intermittent errors in logs
- Check if session stickiness enabled

**Typical fix**
- Fix failing pods causing removal from endpoints
- Increase health check tolerances if too strict
- Scale down bad pods and investigate
- Fix network or application stability issues
- Disable session affinity to test

**One-line KT sentence**
"Route works intermittently because some pods are unhealthy and flapping in and out of the endpoint list."

---

## 15. DNS Resolution Failing Inside Pod

**Trigger symptom**
- Cannot resolve service names or external DNS from pod

**What is usually wrong**
- CoreDNS pods not running or failing
- Pod DNS policy misconfigured
- Network policy blocking DNS traffic
- DNS service cluster IP unreachable
- No nameserver in resolv.conf

**Immediate checks (COMMANDS FIRST)**
```bash
oc exec <pod-name> -- cat /etc/resolv.conf
oc exec <pod-name> -- nslookup kubernetes.default
oc get pods -n openshift-dns
oc describe pod <pod-name>
oc get svc -n openshift-dns
oc get networkpolicy
```

**Mental debugging flow**
- Check resolv.conf has correct nameserver
- Verify CoreDNS pods are running
- Test DNS service cluster IP directly
- Check dnsPolicy in pod spec
- Look for network policies blocking UDP 53
- Test external DNS vs cluster DNS separately

**Typical fix**
- Restart CoreDNS pods if failing
- Fix network policies to allow DNS
- Set dnsPolicy to ClusterFirst
- Check kubelet DNS settings
- Verify DNS service endpoints exist

**One-line KT sentence**
"DNS resolution fails because CoreDNS pods are down, network policy blocks port 53, or pod DNS configuration is wrong."

---

## 16. ConfigMap Changes Not Reflected

**Trigger symptom**
- Updated ConfigMap but pod still sees old values

**What is usually wrong**
- ConfigMap mounted as volume takes time to sync
- ConfigMap injected as env vars requires pod restart
- Pod cached old values in memory
- Wrong ConfigMap version referenced
- Application not reloading config automatically

**Immediate checks (COMMANDS FIRST)**
```bash
oc get configmap <configmap-name> -o yaml
oc describe pod <pod-name>
oc exec <pod-name> -- cat /path/to/mounted/config
oc exec <pod-name> -- env | grep <VAR>
oc rollout restart deployment/<deployment-name>
```

**Mental debugging flow**
- Check how ConfigMap is consumed (env or volume)
- If env vars, pod restart is required
- If volume, wait for kubelet sync period
- Verify correct ConfigMap is referenced
- Check application reads config dynamically
- Check mounted file content vs ConfigMap

**Typical fix**
- Restart pods to pickup env var changes
- Wait 30-60 seconds for volume mounted ConfigMap sync
- Trigger deployment rollout restart
- Fix application to reload config on change
- Use immutable ConfigMap pattern with new names

**One-line KT sentence**
"ConfigMap changes not reflected because environment variables require pod restart, while volume mounts take time to sync."

---

## 17. Secret Not Mounted / Injected

**Trigger symptom**
- Pod cannot access secret data via mount or env var

**What is usually wrong**
- Secret does not exist in namespace
- Secret name typo in pod spec
- Secret not linked to service account
- Wrong key name in secret reference
- Permission issues with secret access

**Immediate checks (COMMANDS FIRST)**
```bash
oc get secret -n <namespace>
oc describe secret <secret-name>
oc describe pod <pod-name>
oc exec <pod-name> -- ls /path/to/secret
oc exec <pod-name> -- env | grep SECRET
oc get pod <pod-name> -o yaml | grep -A 10 secret
```

**Mental debugging flow**
- Verify secret exists in same namespace
- Check secret name matches pod spec exactly
- Look at pod Events for mount errors
- Test if secret file or env var is accessible
- Check secret keys match references
- Verify service account has access

**Typical fix**
- Create missing secret
- Fix secret name or key in pod spec
- Move or copy secret to correct namespace
- Fix RBAC if permission denied
- Recreate pod after fixing secret

**One-line KT sentence**
"Secret not accessible because it does not exist, name is wrong, or pod spec references wrong key name."

---

## 18. PVC Stuck in Pending

**Trigger symptom**
- PersistentVolumeClaim status remains Pending

**What is usually wrong**
- No storage class available
- Storage class cannot provision dynamically
- No PV matches PVC criteria for static provisioning
- Insufficient storage capacity in cluster
- Wrong access mode requested

**Immediate checks (COMMANDS FIRST)**
```bash
oc get pvc
oc describe pvc <pvc-name>
oc get storageclass
oc get pv
oc get events --sort-by='.lastTimestamp'
```

**Mental debugging flow**
- Check Events for provisioning errors
- Verify storage class exists and is default
- Check provisioner logs if dynamic
- Look for matching PV if static binding
- Verify requested size available
- Check access mode compatibility

**Typical fix**
- Install or configure storage provisioner
- Create PV manually for static provisioning
- Fix storage class configuration
- Reduce requested storage size
- Change access mode to supported type

**One-line KT sentence**
"PVC stuck Pending because no storage class exists, provisioner failed, or no PV matches the claim criteria."

---

## 19. PVC Mount Failure in Pod

**Trigger symptom**
- Pod stuck in ContainerCreating with PVC mount error

**What is usually wrong**
- PVC not bound to PV
- PV in use by another pod (non-shared mode)
- Node cannot access storage backend
- Permission denied on volume
- Storage backend unavailable

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc get pvc
oc describe pvc <pvc-name>
oc get pv
oc describe pv <pv-name>
oc get events
```

**Mental debugging flow**
- Check PVC status is Bound
- Verify PV shows correct access mode
- Look at Events for mount errors
- Check if PV already attached to node
- Verify storage backend reachable from node
- Check access mode allows multiple pods

**Typical fix**
- Wait for PVC to bind if pending
- Delete other pod using PV if RWO
- Fix storage backend connectivity
- Change access mode to RWX if multi-pod
- Check node and volume zone match

**One-line KT sentence**
"PVC mount fails because volume already in use with ReadWriteOnce, PVC unbound, or storage backend unreachable."

---

## 20. Node NotReady

**Trigger symptom**
- Node shows NotReady status in node list

**What is usually wrong**
- Kubelet service stopped or crashed
- Node out of disk space
- Node out of memory or CPU saturated
- Network connectivity lost to node
- Container runtime failed

**Immediate checks (COMMANDS FIRST)**
```bash
oc get nodes
oc describe node <node-name>
oc adm top nodes
oc debug node/<node-name>
systemctl status kubelet
journalctl -u kubelet -f
df -h
```

**Mental debugging flow**
- Check node conditions in describe output
- Look for DiskPressure or MemoryPressure
- Check kubelet last heartbeat time
- Verify kubelet service is running
- Check disk space on node
- Review kubelet logs for errors

**Typical fix**
- Restart kubelet service
- Clean up disk space on node
- Fix network connectivity issues
- Restart container runtime if failed
- Reboot node if necessary

**One-line KT sentence**
"Node NotReady because kubelet stopped, disk full, network connectivity lost, or node resource exhaustion."

---

## 21. Node DiskPressure / MemoryPressure

**Trigger symptom**
- Node conditions show DiskPressure or MemoryPressure

**What is usually wrong**
- Disk usage above threshold (85% or 90%)
- Memory usage above eviction threshold
- Container logs filling disk
- Unused images consuming space
- No pod eviction despite pressure

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe node <node-name>
oc adm top node <node-name>
oc debug node/<node-name>
df -h
du -sh /var/lib/containers/*
crictl images
docker system df
```

**Mental debugging flow**
- Check node conditions section
- Verify disk usage percentage
- Check memory usage vs threshold
- Look for large log files or images
- Check eviction thresholds in kubelet
- Verify pods being evicted if needed

**Typical fix**
- Clean up unused images and containers
- Rotate or truncate large log files
- Increase disk space on node
- Lower log retention settings
- Adjust eviction thresholds in kubelet

**One-line KT sentence**
"Node under DiskPressure from full filesystem or MemoryPressure from exhausted RAM, triggering pod evictions."

---

## 22. High CPU / Memory Usage in Pod

**Trigger symptom**
- Pod consuming excessive CPU or memory resources

**What is usually wrong**
- Application memory leak
- Inefficient code or infinite loops
- No resource limits allowing unbounded usage
- Sudden traffic spike overwhelming pod
- Background tasks consuming resources

**Immediate checks (COMMANDS FIRST)**
```bash
oc top pod <pod-name>
oc describe pod <pod-name>
oc logs <pod-name>
oc exec <pod-name> -- top
oc exec <pod-name> -- ps aux
oc get pod <pod-name> -o yaml | grep -A 5 resources
```

**Mental debugging flow**
- Check current vs limit usage
- Look at resource limits configuration
- Review application logs for errors
- Check for memory leaks in application
- Determine if legitimate load or bug
- Monitor usage trends over time

**Typical fix**
- Set appropriate resource limits
- Fix application memory leak
- Scale horizontally for traffic
- Optimize inefficient code
- Add resource requests and limits
- Profile application for bottlenecks

**One-line KT sentence**
"High resource usage indicates memory leaks, inefficient code, missing limits, or legitimate load requiring scaling."

---

## 23. Liveness / Readiness Probe Failures

**Trigger symptom**
- Pod restarting from liveness failures or not receiving traffic

**What is usually wrong**
- Probe timeout too short for slow startup
- Probe endpoint does not exist
- Application not ready during probe check
- Too frequent probe intervals
- Probe misconfigured with wrong port or path

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc logs <pod-name>
oc get pod <pod-name> -o yaml | grep -A 10 livenessProbe
oc get pod <pod-name> -o yaml | grep -A 10 readinessProbe
oc exec <pod-name> -- curl localhost:8080/health
oc get events
```

**Mental debugging flow**
- Check Events for probe failure messages
- Review probe configuration in pod spec
- Test probe endpoint manually from inside pod
- Check initialDelaySeconds value
- Verify timeout and period settings
- Look at application startup time

**Typical fix**
- Increase initialDelaySeconds for slow startup
- Increase timeout or failure threshold
- Fix probe endpoint in application
- Correct probe port or path
- Make probe less frequent
- Implement proper health check endpoint

**One-line KT sentence**
"Probe failures occur from too-short timeouts, wrong endpoints, or aggressive settings not matching application startup time."

---

## 24. Permission Denied / SCC Related Issues

**Trigger symptom**
- Pod fails with permission denied or SCC errors

**What is usually wrong**
- Application tries to run as root
- Required capabilities not granted
- SELinux context preventing access
- SCC too restrictive for workload
- Volume mount permission issues

**Immediate checks (COMMANDS FIRST)**
```bash
oc describe pod <pod-name>
oc get pod <pod-name> -o yaml | grep -A 5 securityContext
oc get scc
oc describe scc <scc-name>
oc adm policy who-can use scc <scc-name>
oc get sa
```

**Mental debugging flow**
- Check Events for SCC violation messages
- Verify pod security context settings
- Check which SCC is being used
- Look at service account permissions
- Verify required SCC features
- Check volume ownership and permissions

**Typical fix**
- Grant appropriate SCC to service account
- Use anyuid SCC if must run as root
- Fix application to run as non-root
- Add service account to SCC policy
- Adjust volume fsGroup or ownership
- Use privileged SCC only if absolutely necessary

**One-line KT sentence**
"Permission denied from restrictive SCC preventing root access, missing capabilities, or volume ownership mismatches."

---

## 25. Application Starts Then Exits Immediately

**Trigger symptom**
- Pod starts successfully but exits within seconds

**What is usually wrong**
- Application completes task and exits (batch job)
- Missing command or wrong entrypoint
- Configuration error causing immediate exit
- Application cannot find required files
- No foreground process to keep container running

**Immediate checks (COMMANDS FIRST)**
```bash
oc logs <pod-name>
oc logs <pod-name> --previous
oc describe pod <pod-name>
oc get pod <pod-name> -o yaml | grep -A 5 command
oc get events
```

**Mental debugging flow**
- Read application logs for exit reason
- Check exit code in describe output
- Verify command and args in pod spec
- Check if supposed to run as batch job
- Look for missing environment or config
- Verify entrypoint script exists and is executable

**Typical fix**
- Use Job instead of Deployment for batch work
- Fix command or entrypoint configuration
- Add tail -f or infinite loop if needed
- Fix missing configuration files
- Make entrypoint script executable
- Add proper application startup command

**One-line KT sentence**
"Application exits immediately because it completes quickly (use Job), has wrong command, or configuration prevents startup."

---

## General Troubleshooting Commands Reference

**Essential commands for any scenario**

```bash
oc status
oc get all
oc get events --sort-by='.lastTimestamp'
oc get events --field-selector type=Warning
oc describe <resource> <name>
oc logs <pod-name> --previous
oc logs <pod-name> -f --tail=50
oc exec <pod-name> -- <command>
oc debug node/<node-name>
oc adm top nodes
oc adm top pods
oc get pods -o wide
oc get pod <pod-name> -o yaml
oc explain <resource>
```

**Quick mental model for any incident**

1. Identify the failing component level (Pod, Service, Node, Route)
2. Get current state and events
3. Read logs and describe output
4. Test connectivity and access
5. Check resource limits and quotas
6. Verify configuration and secrets
7. Review recent changes
8. Fix root cause then verify

**Remember**

- Events expire after 1 hour, check immediately
- Previous logs disappear after next restart
- Describe output shows last events and state
- Test from inside cluster before external
- Check labels and selectors match exactly
- Resource limits affect scheduling and runtime
- Security contexts and SCC block many operations
