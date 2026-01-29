# OpenShift Core Resources – High Retrieval Cheat Sheet

This document is designed for **fast recall** and **knowledge transfer (KT)**. It assumes you already understand OpenShift concepts and need quick retrieval for teaching, interviews, or refreshers. Each section provides triggers, definitions, CLI commands, YAML templates, and concrete examples for continuous speaking.

---

## Table of Contents

1. [Left-Side Topic Map (Overview)](#left-side-topic-map-overview)
2. [Projects / Namespaces](#projects--namespaces)
3. [Nodes](#nodes)
4. [Pods](#pods)
5. [Deployments](#deployments)
6. [DeploymentConfigs](#deploymentconfigs)
7. [ReplicaSets](#replicasets)
8. [StatefulSets](#statefulsets)
9. [DaemonSets](#daemonsets)
10. [Jobs / CronJobs](#jobs--cronjobs)
11. [Services (ClusterIP, NodePort, LoadBalancer)](#services-clusterip-nodeport-loadbalancer)
12. [Routes](#routes)
13. [Ingress](#ingress)
14. [ConfigMaps](#configmaps)
15. [Secrets](#secrets)
16. [ServiceAccounts](#serviceaccounts)
17. [Roles & RoleBindings (RBAC)](#roles--rolebindings-rbac)
18. [PersistentVolumeClaims (PVC)](#persistentvolumeclaims-pvc)
19. [StorageClasses](#storageclasses)
20. [BuildConfigs](#buildconfigs)
21. [ImageStreams](#imagestreams)
22. [Templates](#templates)
23. [Operators / OperatorHub](#operators--operatorhub)
24. [Closing Notes](#closing-notes)

---

## Left-Side Topic Map (Overview)

| Topic | What it is (1 line) | Most used oc command |
|-------|---------------------|----------------------|
| Projects / Namespaces | Logical isolation boundary for resources in OpenShift | `oc get projects` |
| Nodes | Physical or virtual machines that run workloads | `oc get nodes` |
| Pods | Smallest runnable unit – wrapper around containers | `oc get pods` |
| Deployments | Declarative way to manage ReplicaSets and Pods (K8s native) | `oc get deployments` |
| DeploymentConfigs | OpenShift-specific deployment with triggers and strategies | `oc get dc` |
| ReplicaSets | Ensures desired number of Pod replicas are running | `oc get rs` |
| StatefulSets | Manages stateful applications with stable network identity | `oc get statefulsets` |
| DaemonSets | Ensures a Pod runs on every (or selected) node | `oc get daemonsets` |
| Jobs / CronJobs | Run tasks to completion once or on a schedule | `oc get jobs` / `oc get cronjobs` |
| Services | Network abstraction for accessing Pods (ClusterIP, NodePort, LoadBalancer) | `oc get svc` |
| Routes | Exposes Services externally with hostname (OpenShift-specific) | `oc get routes` |
| Ingress | K8s-native way to expose HTTP/HTTPS routes | `oc get ingress` |
| ConfigMaps | Store non-sensitive configuration data as key-value pairs | `oc get configmaps` |
| Secrets | Store sensitive data like passwords, tokens, keys | `oc get secrets` |
| ServiceAccounts | Identity for processes running in Pods | `oc get sa` |
| Roles & RoleBindings | RBAC – define and assign permissions within namespaces | `oc get roles` / `oc get rolebindings` |
| PersistentVolumeClaims | Request for storage by Pods | `oc get pvc` |
| StorageClasses | Define types of storage with provisioners | `oc get sc` |
| BuildConfigs | Defines source-to-image (S2I) build process | `oc get bc` |
| ImageStreams | Tracks and manages container images in OpenShift | `oc get is` |
| Templates | Reusable parameterized definitions of resources | `oc get templates` |
| Operators / OperatorHub | Automate deployment, scaling, and management of applications | `oc get operators` |

---

## Projects / Namespaces

**Primary Trigger:** "Projects are OpenShift's way of isolating resources – namespaces with additional RBAC and quotas."

### 1. Simple Definition

- A **Project** is OpenShift's extension of a Kubernetes namespace
- Provides logical isolation for resources (Pods, Services, etc.)
- Includes built-in RBAC, resource quotas, and network policies

### 2. When to Use / Why it Exists

- Isolate workloads for different teams, applications, or environments
- Apply resource limits and quotas per project
- Simplify multi-tenancy with access controls
- Default context for all `oc` commands

### 3. Important CLI Commands

```bash
# list all projects
oc get projects

# create a new project
oc new-project my-project

# switch to a project
oc project my-project

# delete a project
oc delete project my-project

# describe project details
oc describe project my-project
```

### 4. Minimal YAML Template

```yaml
apiVersion: project.openshift.io/v1
kind: Project
metadata:
  name: my-project
  annotations:
    openshift.io/description: "My application project"
    openshift.io/display-name: "My Project"
```

### 5. One Short Example / Scenario

- **Scenario:** Create a dev environment for a team
- Run `oc new-project team-dev` to create isolated workspace
- Deploy application resources only visible within this project
- Apply resource quotas to limit CPU/memory usage

### 6. Closing Line Trigger

"In short, Projects provide isolated workspaces with built-in RBAC and quota management in OpenShift."

---

## Nodes

**Primary Trigger:** "Nodes are the worker machines – physical or virtual – where Pods actually execute."

### 1. Simple Definition

- A **Node** is a worker machine in the OpenShift cluster (VM or bare metal)
- Runs the container runtime (CRI-O), kubelet, and kube-proxy
- Managed by the control plane; schedules and runs Pods

### 2. When to Use / Why it Exists

- Provides compute resources (CPU, memory, storage) for workloads
- Can be labeled and selected for specific workload placement
- Scale cluster capacity by adding or removing nodes
- Monitor node health, capacity, and resource utilization

### 3. Important CLI Commands

```bash
# list all nodes
oc get nodes

# describe node details
oc describe node <node-name>

# label a node
oc label node <node-name> env=prod

# check node resource usage
oc adm top nodes

# cordon a node (mark unschedulable)
oc adm cordon <node-name>

# drain a node (evict pods)
oc adm drain <node-name>
```

### 4. Minimal YAML Template

```yaml
# Nodes are typically managed by the cluster, not created manually
# But you can view node configuration:
apiVersion: v1
kind: Node
metadata:
  name: worker-node-1
  labels:
    node-role.kubernetes.io/worker: ""
spec:
  taints:
    - effect: NoSchedule
      key: maintenance
      value: "true"
```

### 5. One Short Example / Scenario

- **Scenario:** Perform maintenance on a node
- Run `oc adm cordon worker-1` to prevent new Pods from scheduling
- Run `oc adm drain worker-1` to safely evict existing Pods
- After maintenance, run `oc adm uncordon worker-1` to resume scheduling

### 6. Closing Line Trigger

"In short, Nodes provide the physical infrastructure where all containerized workloads run in OpenShift."

---

## Pods

**Primary Trigger:** "Smallest runnable unit – wrapper around one or more containers on a node."

### 1. Simple Definition

- A **Pod** is the smallest deployable unit in OpenShift/Kubernetes
- Contains one or more containers that share network and storage
- Ephemeral – designed to be replaced, not repaired

### 2. When to Use / Why it Exists

- Group tightly coupled containers (e.g., app + sidecar logging)
- Basic execution unit for all workloads
- Typically managed by higher-level controllers (Deployments, StatefulSets)
- Direct Pod creation is mainly for testing or debugging

### 3. Important CLI Commands

```bash
# list pods
oc get pods

# describe pod details
oc describe pod <pod-name>

# create pod from yaml
oc apply -f pod.yaml

# delete a pod
oc delete pod <pod-name>

# get pod logs
oc logs <pod-name>

# execute command in pod
oc exec -it <pod-name> -- /bin/bash

# port forward to pod
oc port-forward <pod-name> 8080:8080
```

### 4. Minimal YAML Template

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  labels:
    app: nginx
spec:
  containers:
    - name: app
      image: nginx:latest
      ports:
        - containerPort: 80
```

### 5. One Short Example / Scenario

- **Scenario:** Quickly test nginx deployment
- Create `pod.yaml` with nginx container spec
- Run `oc apply -f pod.yaml` to create the Pod
- Access with `oc port-forward example-pod 8080:80` and test locally

### 6. Closing Line Trigger

"In short, Pods are the basic execution units where containers actually run in OpenShift."

---

## Deployments

**Primary Trigger:** "Kubernetes-native declarative way to manage ReplicaSets and rolling updates."

### 1. Simple Definition

- A **Deployment** manages ReplicaSets and provides declarative Pod updates
- Standard Kubernetes resource (not OpenShift-specific)
- Handles rolling updates, rollbacks, and scaling automatically

### 2. When to Use / Why it Exists

- Deploy stateless applications with automatic updates
- Perform rolling updates with zero downtime
- Rollback to previous versions easily
- Preferred over DeploymentConfigs in modern OpenShift (4.x+)

### 3. Important CLI Commands

```bash
# list deployments
oc get deployments

# describe deployment
oc describe deployment <name>

# create deployment
oc create deployment nginx --image=nginx:latest

# scale deployment
oc scale deployment <name> --replicas=3

# rollout status
oc rollout status deployment/<name>

# rollback to previous version
oc rollout undo deployment/<name>

# edit deployment
oc edit deployment <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
```

### 5. One Short Example / Scenario

- **Scenario:** Deploy and update a web application
- Create Deployment with `replicas: 3` for high availability
- Update image version: `oc set image deployment/nginx nginx=nginx:1.22`
- Monitor rollout: `oc rollout status deployment/nginx`
- If issues arise, rollback: `oc rollout undo deployment/nginx`

### 6. Closing Line Trigger

"In short, Deployments provide declarative, automated management of stateless applications with built-in rollout capabilities."

---

## DeploymentConfigs

**Primary Trigger:** "OpenShift-specific deployment resource with triggers, hooks, and custom strategies."

### 1. Simple Definition

- A **DeploymentConfig** is OpenShift's original deployment resource
- Supports image change triggers and configuration change triggers
- Provides deployment hooks (pre, mid, post) and custom strategies

### 2. When to Use / Why it Exists

- Automatic redeployment when ImageStream tags update
- Custom deployment strategies (Rolling, Recreate, Custom)
- Lifecycle hooks for complex deployment workflows
- Legacy support – Deployments are now preferred in OpenShift 4.x+

### 3. Important CLI Commands

```bash
# list deployment configs
oc get dc

# describe deployment config
oc describe dc <name>

# create deployment config
oc create dc myapp --image=myapp:latest

# trigger new deployment
oc rollout latest dc/<name>

# scale deployment config
oc scale dc <name> --replicas=3

# rollback deployment config
oc rollback dc/<name>

# view deployment history
oc rollout history dc/<name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: myapp-dc
spec:
  replicas: 2
  selector:
    app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
  triggers:
    - type: ConfigChange
    - type: ImageChange
      imageChangeParams:
        automatic: true
        containerNames:
          - myapp
        from:
          kind: ImageStreamTag
          name: myapp:latest
```

### 5. One Short Example / Scenario

- **Scenario:** Auto-redeploy when new image is built
- Create ImageStream: `oc create imagestream myapp`
- Create DeploymentConfig with ImageChange trigger
- When new build pushes to ImageStream, deployment triggers automatically
- No manual intervention needed for continuous deployment

### 6. Closing Line Trigger

"In short, DeploymentConfigs offer OpenShift-native deployment automation with triggers and hooks, though Deployments are now preferred."

---

## ReplicaSets

**Primary Trigger:** "Ensures the desired number of identical Pod replicas are always running."

### 1. Simple Definition

- A **ReplicaSet** maintains a stable set of replica Pods
- Ensures specified number of Pods are running at all times
- Typically managed by Deployments – rarely created directly

### 2. When to Use / Why it Exists

- Guarantee availability through Pod replication
- Self-healing – replaces failed Pods automatically
- Scale applications horizontally
- Usually created and managed by Deployment controllers

### 3. Important CLI Commands

```bash
# list replica sets
oc get rs

# describe replica set
oc describe rs <name>

# scale replica set (not recommended, use Deployment)
oc scale rs <name> --replicas=5

# delete replica set
oc delete rs <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:1.0
```

### 5. One Short Example / Scenario

- **Scenario:** Understand Deployment internals
- Create Deployment – observe auto-created ReplicaSet
- Run `oc get rs` to see ReplicaSet managing Pods
- Delete a Pod – ReplicaSet immediately creates replacement
- Scale Deployment – ReplicaSet adjusts Pod count accordingly

### 6. Closing Line Trigger

"In short, ReplicaSets ensure desired Pod replicas run continuously, acting as the engine behind Deployments."

---

## StatefulSets

**Primary Trigger:** "Manages stateful applications with stable network identity and persistent storage."

### 1. Simple Definition

- A **StatefulSet** manages Pods with unique, stable identities
- Each Pod gets a persistent hostname and storage
- Maintains order during scaling and updates

### 2. When to Use / Why it Exists

- Deploy databases (MySQL, PostgreSQL, MongoDB)
- Applications requiring stable network identity
- Workloads needing persistent storage tied to specific Pods
- Ordered deployment, scaling, and updates

### 3. Important CLI Commands

```bash
# list stateful sets
oc get statefulsets

# describe stateful set
oc describe statefulset <name>

# scale stateful set
oc scale statefulset <name> --replicas=5

# delete stateful set (keeps PVCs)
oc delete statefulset <name>

# delete stateful set and PVCs
oc delete statefulset <name> --cascade=orphan
```

### 4. Minimal YAML Template

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

### 5. One Short Example / Scenario

- **Scenario:** Deploy a 3-node MySQL cluster
- Create StatefulSet with `replicas: 3` and volumeClaimTemplates
- Pods named `mysql-0`, `mysql-1`, `mysql-2` with unique storage
- Scale down: Pods removed in reverse order (2, 1, 0)
- PVCs persist even if StatefulSet is deleted for data safety

### 6. Closing Line Trigger

"In short, StatefulSets provide stable identities and persistent storage for applications requiring statefulness."

---

## DaemonSets

**Primary Trigger:** "Ensures every node (or selected nodes) runs exactly one copy of a Pod."

### 1. Simple Definition

- A **DaemonSet** runs one Pod per node automatically
- Useful for node-level services like monitoring, logging, networking
- New nodes automatically get the DaemonSet Pod

### 2. When to Use / Why it Exists

- Deploy cluster-wide agents (Fluentd, Prometheus node exporter)
- Run networking plugins (CNI) or storage daemons
- Monitoring and logging infrastructure
- Node-level system utilities

### 3. Important CLI Commands

```bash
# list daemon sets
oc get daemonsets

# describe daemon set
oc describe daemonset <name>

# create daemon set from yaml
oc apply -f daemonset.yaml

# delete daemon set
oc delete daemonset <name>

# view daemon set status
oc rollout status daemonset/<name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluentd:v1.14
          volumeMounts:
            - name: varlog
              mountPath: /var/log
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
```

### 5. One Short Example / Scenario

- **Scenario:** Deploy logging agent across all cluster nodes
- Create DaemonSet for Fluentd to collect logs from every node
- Each node automatically runs one Fluentd Pod
- Add new node to cluster – Fluentd Pod deployed automatically
- Ensures comprehensive log collection without manual intervention

### 6. Closing Line Trigger

"In short, DaemonSets ensure node-level services run on every node for cluster-wide functionality."

---

## Jobs / CronJobs

**Primary Trigger:** "Run tasks to completion once (Job) or on a schedule (CronJob)."

### 1. Simple Definition

- A **Job** creates Pods that run to completion, then stop
- A **CronJob** schedules Jobs to run at specified times
- Designed for batch processing, backups, or periodic tasks

### 2. When to Use / Why it Exists

- Run one-time tasks (database migrations, data imports)
- Schedule periodic tasks (backups, reports, cleanups)
- Batch processing workflows
- Retries on failure until success or max attempts

### 3. Important CLI Commands

```bash
# list jobs
oc get jobs

# list cron jobs
oc get cronjobs

# describe job
oc describe job <name>

# create job from yaml
oc apply -f job.yaml

# manually trigger a cron job
oc create job --from=cronjob/<name> <job-name>

# delete completed jobs
oc delete job <name>

# view job logs
oc logs job/<name>
```

### 4. Minimal YAML Template

**Job:**
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-import
spec:
  template:
    spec:
      containers:
        - name: import
          image: myapp:latest
          command: ["python", "import_data.py"]
      restartPolicy: OnFailure
  backoffLimit: 3
```

**CronJob:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-db
spec:
  schedule: "0 2 * * *"  # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: backup
              image: postgres:14
              command: ["pg_dump", "-h", "db", "-U", "user"]
          restartPolicy: OnFailure
```

### 5. One Short Example / Scenario

- **Scenario:** Daily database backup at 2 AM
- Create CronJob with schedule `0 2 * * *`
- CronJob creates a Job every day at 2 AM
- Job Pod runs backup script and completes
- Failed backups retry automatically up to `backoffLimit`

### 6. Closing Line Trigger

"In short, Jobs run tasks to completion while CronJobs schedule them periodically for automated workflows."

---

## Services (ClusterIP, NodePort, LoadBalancer)

**Primary Trigger:** "Network abstraction providing stable endpoint for accessing Pods."

### 1. Simple Definition

- A **Service** exposes Pods on a stable IP and DNS name
- Three main types: ClusterIP (internal), NodePort (node-level), LoadBalancer (external)
- Load balances traffic across matching Pods

### 2. When to Use / Why it Exists

- Provide stable endpoint despite Pod IP changes
- Load balance traffic across replica Pods
- Enable service discovery via DNS
- Expose applications internally (ClusterIP) or externally (NodePort/LoadBalancer)

### 3. Important CLI Commands

```bash
# list services
oc get svc

# describe service
oc describe svc <name>

# create service (expose deployment)
oc expose deployment <name> --port=80 --target-port=8080

# create service from yaml
oc apply -f service.yaml

# delete service
oc delete svc <name>

# get service endpoints
oc get endpoints <svc-name>
```

### 4. Minimal YAML Template

**ClusterIP (default):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**NodePort:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

**LoadBalancer:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 5. One Short Example / Scenario

- **Scenario:** Expose a web app to other services in cluster
- Create Deployment with label `app: webapp`
- Create ClusterIP Service selecting `app: webapp`
- Other Pods access via `http://webapp-service:80`
- Service automatically load balances across all webapp Pods

### 6. Closing Line Trigger

"In short, Services provide stable networking endpoints for dynamic Pod sets with built-in load balancing."

---

## Routes

**Primary Trigger:** "OpenShift-specific resource that exposes Services externally with hostname and SSL."

### 1. Simple Definition

- A **Route** exposes a Service via external hostname (URL)
- OpenShift-native alternative to Ingress
- Supports TLS termination, path-based routing, and load balancing

### 2. When to Use / Why it Exists

- Make applications accessible from outside the cluster
- Provide user-friendly URLs (e.g., `myapp.example.com`)
- Built-in SSL/TLS certificate management
- More feature-rich than basic Kubernetes Ingress in OpenShift

### 3. Important CLI Commands

```bash
# list routes
oc get routes

# describe route
oc describe route <name>

# expose service with route
oc expose service <svc-name>

# create route with custom hostname
oc expose service <svc-name> --hostname=myapp.example.com

# create secure route (TLS)
oc create route edge --service=<svc-name> --hostname=myapp.example.com

# delete route
oc delete route <name>
```

### 4. Minimal YAML Template

**HTTP Route:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp-route
spec:
  host: myapp.example.com
  to:
    kind: Service
    name: myapp-service
  port:
    targetPort: 8080
```

**HTTPS Route (Edge Termination):**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: myapp-secure-route
spec:
  host: myapp.example.com
  to:
    kind: Service
    name: myapp-service
  port:
    targetPort: 8080
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

### 5. One Short Example / Scenario

- **Scenario:** Make internal web app publicly accessible
- Application running as Service `webapp-service` on port 80
- Run `oc expose service webapp-service` to create Route
- OpenShift assigns hostname like `webapp-myproject.apps.cluster.com`
- Users access via browser using the generated URL

### 6. Closing Line Trigger

"In short, Routes expose Services externally with friendly URLs and built-in TLS support in OpenShift."

---

## Ingress

**Primary Trigger:** "Kubernetes-native way to expose HTTP/HTTPS routes to Services."

### 1. Simple Definition

- An **Ingress** manages external access to Services via HTTP/HTTPS
- Standard Kubernetes resource (not OpenShift-specific)
- Requires an Ingress controller to function

### 2. When to Use / Why it Exists

- Portable across Kubernetes platforms (not OpenShift-specific)
- Path-based routing to multiple Services
- Name-based virtual hosting
- OpenShift supports both Ingress and Routes (Routes preferred)

### 3. Important CLI Commands

```bash
# list ingress resources
oc get ingress

# describe ingress
oc describe ingress <name>

# create ingress from yaml
oc apply -f ingress.yaml

# delete ingress
oc delete ingress <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
```

### 5. One Short Example / Scenario

- **Scenario:** Route multiple paths to different Services
- `/api` → api-service, `/web` → web-service
- Create Ingress with multiple path rules
- Single hostname serves both APIs and web UI
- Ingress controller routes traffic based on path

### 6. Closing Line Trigger

"In short, Ingress provides Kubernetes-standard HTTP routing, though OpenShift Routes offer more native features."

---

## ConfigMaps

**Primary Trigger:** "Store non-sensitive configuration data as key-value pairs for Pods."

### 1. Simple Definition

- A **ConfigMap** stores configuration data separately from container images
- Data stored as key-value pairs or full file contents
- Injected into Pods as environment variables or mounted as files

### 2. When to Use / Why it Exists

- Externalize configuration from application code
- Enable same image to run in different environments
- Store config files, environment-specific settings
- Non-sensitive data only (use Secrets for passwords)

### 3. Important CLI Commands

```bash
# list config maps
oc get configmaps

# describe config map
oc describe configmap <name>

# create from literal values
oc create configmap app-config --from-literal=DB_HOST=localhost

# create from file
oc create configmap app-config --from-file=config.properties

# create from yaml
oc apply -f configmap.yaml

# delete config map
oc delete configmap <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "postgres.example.com"
  DB_PORT: "5432"
  LOG_LEVEL: "info"
  config.properties: |
    app.name=MyApp
    app.version=1.0
```

**Usage in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: app
      image: myapp:latest
      envFrom:
        - configMapRef:
            name: app-config
```

### 5. One Short Example / Scenario

- **Scenario:** Configure database connection per environment
- Create ConfigMap for dev: `DB_HOST=dev-db.local`
- Create ConfigMap for prod: `DB_HOST=prod-db.local`
- Same application image reads from ConfigMap
- No code changes needed between environments

### 6. Closing Line Trigger

"In short, ConfigMaps decouple configuration from container images enabling environment-specific deployments."

---

## Secrets

**Primary Trigger:** "Store and manage sensitive data like passwords, tokens, and SSH keys."

### 1. Simple Definition

- A **Secret** stores sensitive information (passwords, keys, certificates)
- Base64-encoded (not encrypted by default)
- Injected into Pods as environment variables or mounted files

### 2. When to Use / Why it Exists

- Store credentials securely separate from code
- Manage TLS certificates and SSH keys
- Provide tokens and API keys to applications
- More secure than hardcoding or using ConfigMaps

### 3. Important CLI Commands

```bash
# list secrets
oc get secrets

# describe secret (data hidden by default)
oc describe secret <name>

# create secret from literal
oc create secret generic db-secret --from-literal=password=mypass

# create secret from file
oc create secret generic ssh-key --from-file=id_rsa=~/.ssh/id_rsa

# create TLS secret
oc create secret tls tls-secret --cert=tls.crt --key=tls.key

# decode secret
oc get secret <name> -o jsonpath='{.data.password}' | base64 -d

# delete secret
oc delete secret <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=     # base64 encoded "admin"
  password: cGFzczEyMzQ=  # base64 encoded "pass1234"
```

**Usage in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: app
      image: myapp:latest
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

### 5. One Short Example / Scenario

- **Scenario:** Provide database credentials to application
- Create Secret with username and password
- Reference Secret in Deployment as environment variable
- Application reads `DB_PASSWORD` from environment
- Rotate password by updating Secret (Pods auto-reload)

### 6. Closing Line Trigger

"In short, Secrets securely store sensitive data and inject it into Pods without exposing it in code."

---

## ServiceAccounts

**Primary Trigger:** "Identity for processes running inside Pods to interact with the API."

### 1. Simple Definition

- A **ServiceAccount** provides identity for Pods to access OpenShift API
- Every Pod has an associated ServiceAccount (default if not specified)
- Used for RBAC – controlling what Pods can do

### 2. When to Use / Why it Exists

- Grant Pods specific API permissions
- Authenticate automated processes or controllers
- Separate permissions for different applications
- Required for CI/CD pipelines running in-cluster

### 3. Important CLI Commands

```bash
# list service accounts
oc get sa

# describe service account
oc describe sa <name>

# create service account
oc create sa my-sa

# delete service account
oc delete sa <name>

# grant permissions to service account
oc adm policy add-role-to-user edit -z my-sa
```

### 4. Minimal YAML Template

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```

**Usage in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  serviceAccountName: my-service-account
  containers:
    - name: app
      image: myapp:latest
```

### 5. One Short Example / Scenario

- **Scenario:** CI/CD pipeline needs to create Pods
- Create ServiceAccount `pipeline-sa`
- Grant edit permissions: `oc adm policy add-role-to-user edit -z pipeline-sa`
- Pipeline Pods use this ServiceAccount
- Limited to edit permissions, not cluster-admin

### 6. Closing Line Trigger

"In short, ServiceAccounts provide identity and RBAC-based authorization for Pods accessing the OpenShift API."

---

## Roles & RoleBindings (RBAC)

**Primary Trigger:** "Define and assign permissions within namespaces using role-based access control."

### 1. Simple Definition

- A **Role** defines a set of permissions (verbs on resources)
- A **RoleBinding** assigns a Role to users, groups, or ServiceAccounts
- Namespace-scoped (ClusterRole/ClusterRoleBinding for cluster-wide)

### 2. When to Use / Why it Exists

- Implement least-privilege access control
- Grant specific permissions to users or ServiceAccounts
- Separate concerns between different teams or applications
- Secure multi-tenant environments

### 3. Important CLI Commands

```bash
# list roles
oc get roles

# list role bindings
oc get rolebindings

# describe role
oc describe role <name>

# create role
oc create role pod-reader --verb=get,list --resource=pods

# create role binding
oc create rolebinding read-pods --role=pod-reader --user=john

# grant admin to user in project
oc adm policy add-role-to-user admin john -n myproject

# remove role from user
oc adm policy remove-role-from-user admin john
```

### 4. Minimal YAML Template

**Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

**RoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
  - kind: User
    name: john
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 5. One Short Example / Scenario

- **Scenario:** Give developer read-only access to Pods
- Create Role `pod-reader` with `get`, `list`, `watch` verbs
- Create RoleBinding assigning Role to user `jane`
- Jane can now view Pods but cannot create or delete them
- Enforces least-privilege principle for security

### 6. Closing Line Trigger

"In short, Roles define permissions while RoleBindings assign them, implementing fine-grained access control."

---

## PersistentVolumeClaims (PVC)

**Primary Trigger:** "Request for storage that Pods can use for persistent data."

### 1. Simple Definition

- A **PVC** is a storage request by a Pod for persistent volumes
- Abstracts storage details from Pods
- Bound to a PersistentVolume (PV) automatically or dynamically provisioned

### 2. When to Use / Why it Exists

- Store data that persists beyond Pod lifecycle
- Database storage, file uploads, logs
- Decouples storage requests from storage implementation
- Enables dynamic provisioning via StorageClasses

### 3. Important CLI Commands

```bash
# list persistent volume claims
oc get pvc

# describe pvc
oc describe pvc <name>

# create pvc from yaml
oc apply -f pvc.yaml

# delete pvc
oc delete pvc <name>

# check bound volume
oc get pvc <name> -o jsonpath='{.spec.volumeName}'
```

### 4. Minimal YAML Template

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

**Usage in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: app
      image: myapp:latest
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: data-pvc
```

### 5. One Short Example / Scenario

- **Scenario:** Database needs persistent storage
- Create PVC requesting 50Gi of storage
- StorageClass automatically provisions PV
- Database Pod mounts PVC to `/var/lib/mysql`
- Data survives Pod restarts and rescheduling

### 6. Closing Line Trigger

"In short, PVCs provide persistent storage requests that outlive Pod lifecycles for stateful applications."

---

## StorageClasses

**Primary Trigger:** "Define types of storage with dynamic provisioners for automatic PV creation."

### 1. Simple Definition

- A **StorageClass** describes storage "classes" with different characteristics
- Enables dynamic provisioning of PersistentVolumes
- Defines provisioner (AWS EBS, NFS, Ceph, etc.) and parameters

### 2. When to Use / Why it Exists

- Automate PV creation when PVCs are requested
- Offer different storage tiers (SSD vs HDD, fast vs slow)
- Abstract storage backend details from users
- Simplify storage management at scale

### 3. Important CLI Commands

```bash
# list storage classes
oc get sc

# describe storage class
oc describe sc <name>

# create storage class from yaml
oc apply -f storageclass.yaml

# set default storage class
oc patch storageclass <name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# delete storage class
oc delete sc <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### 5. One Short Example / Scenario

- **Scenario:** Provide SSD and HDD storage options
- Create StorageClass `fast-ssd` for high-performance workloads
- Create StorageClass `slow-hdd` for bulk storage
- Developers specify `storageClassName` in PVCs
- Appropriate storage provisioned based on requirements

### 6. Closing Line Trigger

"In short, StorageClasses enable dynamic storage provisioning with configurable performance and cost characteristics."

---

## BuildConfigs

**Primary Trigger:** "Defines the source-to-image (S2I) build process for creating container images."

### 1. Simple Definition

- A **BuildConfig** defines how to build container images in OpenShift
- Supports multiple strategies: Source-to-Image (S2I), Docker, Custom
- Automates building from source code repositories

### 2. When to Use / Why it Exists

- Automate image builds from source code (Git repos)
- Convert source code to runnable container images
- Trigger builds on code changes or webhooks
- OpenShift's built-in CI/CD capability

### 3. Important CLI Commands

```bash
# list build configs
oc get bc

# describe build config
oc describe bc <name>

# start a new build
oc start-build <bc-name>

# follow build logs
oc logs -f bc/<bc-name>

# create build config from source
oc new-app nodejs~https://github.com/user/repo

# delete build config
oc delete bc <name>

# list builds
oc get builds
```

### 4. Minimal YAML Template

```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: myapp-build
spec:
  source:
    type: Git
    git:
      uri: https://github.com/user/myapp.git
      ref: main
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: nodejs:16
  output:
    to:
      kind: ImageStreamTag
      name: myapp:latest
  triggers:
    - type: ConfigChange
    - type: GitHub
      github:
        secret: webhook-secret
```

### 5. One Short Example / Scenario

- **Scenario:** Build Node.js app from GitHub repo
- Run `oc new-app nodejs~https://github.com/user/app.git`
- OpenShift creates BuildConfig, ImageStream, DeploymentConfig
- S2I builder compiles source into container image
- Automatic webhook triggers rebuild on Git push

### 6. Closing Line Trigger

"In short, BuildConfigs automate container image creation from source code using OpenShift's integrated build system."

---

## ImageStreams

**Primary Trigger:** "Tracks and manages container image tags with automatic deployment triggers."

### 1. Simple Definition

- An **ImageStream** is OpenShift's abstraction for container images
- Tracks image tags and references (external or internal registry)
- Triggers deployments when new image versions are pushed

### 2. When to Use / Why it Exists

- Centralize image management and versioning
- Decouple image sources from deployments
- Enable automatic redeployment on image updates
- Support image promotion across environments (dev → staging → prod)

### 3. Important CLI Commands

```bash
# list image streams
oc get is

# describe image stream
oc describe is <name>

# view image stream tags
oc get istag

# import external image
oc import-image myapp:latest --from=docker.io/user/myapp:latest --confirm

# tag image for promotion
oc tag myapp:latest myapp:prod

# delete image stream
oc delete is <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: myapp
spec:
  lookupPolicy:
    local: false
  tags:
    - name: latest
      from:
        kind: DockerImage
        name: docker.io/myuser/myapp:latest
      importPolicy:
        scheduled: true
```

### 5. One Short Example / Scenario

- **Scenario:** Continuous deployment on image updates
- BuildConfig outputs to ImageStream `myapp:latest`
- DeploymentConfig watches ImageStream for changes
- New build completes → ImageStream updated → DeploymentConfig triggers
- Zero manual intervention for deployment pipeline

### 6. Closing Line Trigger

"In short, ImageStreams provide intelligent image tracking with automated deployment triggers for CI/CD workflows."

---

## Templates

**Primary Trigger:** "Reusable parameterized definitions for deploying multi-resource applications."

### 1. Simple Definition

- A **Template** is a parameterized collection of OpenShift resources
- Define once, reuse with different parameter values
- Simplifies complex application deployments

### 2. When to Use / Why it Exists

- Deploy complete applications with multiple resources (Deployment, Service, Route, etc.)
- Provide reusable blueprints for common patterns
- Enable self-service application provisioning
- Support environment-specific customization via parameters

### 3. Important CLI Commands

```bash
# list templates
oc get templates

# describe template
oc describe template <name>

# process template (generate resources)
oc process -f template.yaml -p APP_NAME=myapp

# create from template
oc new-app --template=<template-name> -p PARAM1=value1

# process and apply template
oc process -f template.yaml | oc apply -f -

# delete template
oc delete template <name>
```

### 4. Minimal YAML Template

```yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: webapp-template
parameters:
  - name: APP_NAME
    description: Application name
    required: true
  - name: IMAGE_TAG
    description: Image tag
    value: latest
objects:
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ${APP_NAME}
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: ${APP_NAME}
      template:
        metadata:
          labels:
            app: ${APP_NAME}
        spec:
          containers:
            - name: ${APP_NAME}
              image: ${APP_NAME}:${IMAGE_TAG}
  - apiVersion: v1
    kind: Service
    metadata:
      name: ${APP_NAME}
    spec:
      selector:
        app: ${APP_NAME}
      ports:
        - port: 80
          targetPort: 8080
```

### 5. One Short Example / Scenario

- **Scenario:** Standardize multi-tier app deployment
- Create Template with Deployment, Service, Route, ConfigMap
- Define parameters: `APP_NAME`, `DB_HOST`, `REPLICAS`
- Developers run: `oc new-app --template=webapp -p APP_NAME=myapp`
- Complete application stack deployed with one command

### 6. Closing Line Trigger

"In short, Templates provide reusable, parameterized blueprints for deploying complete application stacks consistently."

---

## Operators / OperatorHub

**Primary Trigger:** "Automate complex application lifecycle management using Kubernetes-native patterns."

### 1. Simple Definition

- An **Operator** is a method of packaging, deploying, and managing applications
- Extends Kubernetes API with Custom Resources (CRDs)
- Codifies operational knowledge (installation, upgrades, backups, recovery)

### 2. When to Use / Why it Exists

- Automate Day 2 operations (upgrades, backups, scaling)
- Deploy complex stateful applications (databases, message queues)
- Standardize application management across clusters
- Leverage community-maintained operators from OperatorHub

### 3. Important CLI Commands

```bash
# list installed operators
oc get operators

# list available operators in catalog
oc get packagemanifests

# list operator subscriptions
oc get subscriptions

# list custom resource definitions
oc get crds

# install operator via subscription
oc apply -f operator-subscription.yaml

# list operator-created custom resources
oc get <custom-resource-kind>

# describe operator
oc describe csv <cluster-service-version-name>
```

### 4. Minimal YAML Template

**OperatorGroup:**
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: my-operatorgroup
  namespace: my-namespace
spec:
  targetNamespaces:
    - my-namespace
```

**Subscription:**
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: my-operator
  namespace: my-namespace
spec:
  channel: stable
  name: my-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
```

### 5. One Short Example / Scenario

- **Scenario:** Deploy PostgreSQL using an Operator
- Search OperatorHub: find PostgreSQL Operator
- Install via Subscription → Operator deployed
- Create custom resource: `kind: PostgreSQL` with desired config
- Operator provisions database, handles backups, manages upgrades

### 6. Closing Line Trigger

"In short, Operators automate complex application management by extending OpenShift with application-specific intelligence."

---

## Closing Notes

This cheat sheet covers the **core OpenShift resources** you need for fast retrieval and knowledge transfer. Each section follows a consistent pattern:

- **Primary Trigger** to anchor your recall
- **Simple Definition** for clarity
- **When to Use** for context
- **CLI Commands** for hands-on practice
- **YAML Templates** for implementation
- **Example Scenarios** for practical understanding
- **Closing Trigger** to wrap up smoothly

Use this document to:
- Revise before interviews or teaching sessions
- Quickly look up commands and YAML syntax
- Speak continuously on any topic with confidence
- Transfer knowledge effectively to team members

**Pro Tips:**
- Practice speaking each section out loud using the triggers
- Combine multiple topics for holistic explanations (e.g., Deployments + Services + Routes)
- Keep this doc open during KT sessions for reference
- Update with your own notes and real-world examples

---

**End of OpenShift Core Resources – High Retrieval Cheat Sheet**
