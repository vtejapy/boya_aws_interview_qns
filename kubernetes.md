# Kubernetes Interview Questions & Detailed Answers - 5 Years Experience

## Section 1: Core Architecture & Components

### 1. Explain the Kubernetes control plane components and what happens if each one fails.

**Answer:**

**API Server (kube-apiserver):**
- Acts as the front-end for the Kubernetes control plane
- Handles all REST operations and validates/configures API objects
- Stores state in etcd and serves as the communication hub
- **If it fails:** Cluster becomes read-only, no new deployments, scaling, or updates possible
- **Mitigation:** Run multiple API server instances behind a load balancer

**etcd:**
- Distributed key-value store that holds all cluster state
- Stores configuration data, secrets, and cluster information
- **If it fails:** Entire cluster becomes unusable, data loss risk
- **Mitigation:** Run etcd cluster with 3 or 5 nodes, regular backups

**Scheduler (kube-scheduler):**
- Assigns pods to nodes based on resource requirements and constraints
- **If it fails:** New pods remain in Pending state, existing pods unaffected
- **Mitigation:** Multiple scheduler instances with leader election

**Controller Manager (kube-controller-manager):**
- Runs various controllers (Deployment, ReplicaSet, Service, etc.)
- **If it fails:** Existing workloads continue but no automatic healing or scaling
- **Mitigation:** Multiple instances with leader election

**Cloud Controller Manager:**
- Integrates with cloud provider APIs
- **If it fails:** Load balancers and persistent volumes may not provision
- **Mitigation:** Multiple instances, graceful degradation

### 2. Describe the complete lifecycle of a pod from creation to termination.

**Answer:**

**Creation Phase:**
1. User submits pod manifest to API server
2. API server validates the request (authentication, authorization, admission controllers)
3. Pod object stored in etcd with status "Pending"
4. Scheduler watches for unscheduled pods, selects appropriate node
5. Scheduler updates pod object with nodeName field
6. kubelet on target node notices the pod assignment
7. kubelet pulls container images and creates containers
8. Pod status updates to "Running"

**Running Phase:**
- kubelet performs health checks (liveness, readiness, startup probes)
- Container runtime manages container lifecycle
- kubelet reports pod status back to API server

**Termination Phase:**
1. Delete request sent to API server
2. Pod marked for deletion with grace period (default 30s)
3. SIGTERM sent to containers
4. If containers don't stop, SIGKILL sent after grace period
5. Pod object removed from etcd

### 3. How does the Kubernetes API request flow work?

**Answer:**

**Authentication Phase:**
- Client certificates, bearer tokens, or webhook authentication
- ServiceAccount tokens for in-cluster communication

**Authorization Phase:**
- RBAC (Role-Based Access Control) - most common
- ABAC (Attribute-Based Access Control)
- Webhook authorization
- AlwaysAllow/AlwaysDeny modes

**Admission Control Phase:**
- Mutating admission controllers (modify requests)
- Validating admission controllers (accept/reject requests)
- Examples: PodSecurityPolicy, ResourceQuota, LimitRanger

**Persistence:**
- Valid requests stored in etcd
- Controllers react to changes via watch mechanisms

### 4. Explain etcd backup and restore procedures for production.

**Answer:**

**Backup Procedure:**
```bash
# Create snapshot
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/server.crt \
  --key=/etc/etcd/server.key

# Verify snapshot
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

**Restore Procedure:**
```bash
# Stop all kube-apiserver instances
systemctl stop kube-apiserver

# Restore snapshot on each etcd node
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --name=etcd-1 \
  --initial-cluster=etcd-1=https://10.0.0.1:2380,etcd-2=https://10.0.0.2:2380,etcd-3=https://10.0.0.3:2380 \
  --initial-advertise-peer-urls=https://10.0.0.1:2380 \
  --data-dir=/var/lib/etcd

# Start etcd cluster
# Start kube-apiserver instances
```

**Best Practices:**
- Automated daily backups
- Store backups in different geographical locations
- Test restore procedures regularly
- Monitor backup success/failure

## Section 2: Workloads & Scheduling

### 5. Pod is stuck in Pending state. Describe your troubleshooting approach.

**Answer:**

**Step 1: Check Pod Description**
```bash
kubectl describe pod <pod-name> -n <namespace>
```

**Common Issues to Check:**

**Resource Constraints:**
```yaml
# Check if node has sufficient resources
kubectl describe nodes
kubectl top nodes

# Example resource requirements causing issues
resources:
  requests:
    memory: "64Gi"  # Too high for available nodes
    cpu: "8"
```

**Node Selectors/Affinity:**
```yaml
# Check if nodeSelector matches any nodes
nodeSelector:
  disk-type: "ssd"  # No nodes with this label

# Check node affinity
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: zone
          operator: In
          values: ["us-west-1a"]  # No nodes in this zone
```

**Taints and Tolerations:**
```bash
# Check node taints
kubectl describe node <node-name> | grep Taints

# Ensure pod has proper tolerations
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"
```

**PodDisruptionBudget:**
```yaml
# Check if PDB is blocking scheduling
kubectl get pdb -A
```

**Image Pull Issues:**
```bash
# Check if image exists and is pullable
kubectl describe pod <pod-name> | grep -A 10 Events
```

### 6. Compare Deployment, StatefulSet, and DaemonSet with real-world examples.

**Answer:**

**Deployment:**
- **Use Case:** Stateless applications (web servers, APIs, microservices)
- **Characteristics:** 
  - Pods are interchangeable
  - Rolling updates supported
  - Horizontal scaling
  - No persistent identity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
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

**StatefulSet:**
- **Use Case:** Stateful applications (databases, message queues, distributed systems)
- **Characteristics:**
  - Stable network identity
  - Ordered deployment/scaling
  - Persistent storage per pod
  - Graceful termination

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-cluster
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
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
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

**DaemonSet:**
- **Use Case:** System-level services (log collectors, monitoring agents, network plugins)
- **Characteristics:**
  - One pod per node
  - Automatically scheduled on new nodes
  - Typically for infrastructure services

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### 7. Explain Horizontal Pod Autoscaler (HPA) implementation and limitations.

**Answer:**

**HPA Implementation:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

**Custom Metrics Setup:**
```yaml
# Prometheus adapter for custom metrics
apiVersion: v1
kind: ConfigMap
metadata:
  name: adapter-config
data:
  config.yaml: |
    rules:
    - seriesQuery: 'http_requests_per_second{namespace!="",pod!=""}'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          pod: {resource: "pod"}
      name:
        matches: "^(.*)_per_second"
        as: "${1}_per_second"
      metricsQuery: 'sum(<<.Series>>{<<.LabelMatchers>>}) by (<<.GroupBy>>)'
```

**Limitations:**
1. **Scaling Frequency:** Default cooldown periods prevent rapid scaling
2. **Metrics Delay:** Metrics collection and processing introduce latency
3. **Cold Start:** New pods take time to become ready
4. **Resource Requests Required:** Pods must have resource requests defined
5. **Single Dimension:** Each HPA targets one workload
6. **Cluster Capacity:** Cannot scale beyond available cluster resources

**Best Practices:**
- Set appropriate resource requests and limits
- Use readiness probes to ensure pod availability
- Implement graceful shutdown handling
- Monitor scaling events and adjust thresholds
- Consider Vertical Pod Autoscaler (VPA) for right-sizing

## Section 3: Networking

### 8. Explain Container Network Interface (CNI) and compare popular implementations.

**Answer:**

**CNI Basics:**
CNI is a specification for configuring network interfaces in Linux containers. It defines how container runtime should invoke network plugins to set up networking.

**CNI Workflow:**
1. Container runtime creates network namespace
2. Runtime calls CNI plugin with configuration
3. Plugin configures network interface in namespace
4. Plugin assigns IP address and sets up routing
5. Plugin returns result to runtime

**Popular CNI Implementations:**

**Calico:**
```yaml
# Calico NetworkPolicy example
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Features:**
- BGP-based routing
- Advanced network policies
- eBPF data plane option
- Service mesh integration
- IP-in-IP and VXLAN encapsulation

**Flannel:**
```yaml
# Flannel configuration
net-conf.json: |
  {
    "Network": "10.244.0.0/16",
    "Backend": {
      "Type": "vxlan"
    }
  }
```

**Features:**
- Simple overlay network
- VXLAN, host-gw, UDP backends
- Easy to deploy and manage
- Limited network policy support

**Cilium:**
```yaml
# Cilium L7 Network Policy
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: "l7-rule"
spec:
  endpointSelector:
    matchLabels:
      app: api
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/.*"
```

**Features:**
- eBPF-based data plane
- L3/L4/L7 network policies
- Service mesh capabilities
- Advanced observability
- High performance

### 9. Design network policies for a multi-tier application.

**Answer:**

**Application Architecture:**
- Frontend (React app)
- API Gateway
- Backend Services (User, Order, Payment)
- Database (PostgreSQL)
- Cache (Redis)

**Network Policies:**

**1. Default Deny All:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**2. Frontend Policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: api-gateway
    ports:
    - protocol: TCP
      port: 8080
  - to: []  # DNS resolution
    ports:
    - protocol: UDP
      port: 53
```

**3. API Gateway Policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: api-gateway
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 8080
  - to: []  # DNS and external APIs
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 443
```

**4. Backend Services Policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: api-gateway
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          tier: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - podSelector:
        matchLabels:
          tier: cache
    ports:
    - protocol: TCP
      port: 6379
  - to: []  # DNS
    ports:
    - protocol: UDP
      port: 53
```

**5. Database Policy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 5432
  - from:  # Backup jobs
    - podSelector:
        matchLabels:
          app: backup
    ports:
    - protocol: TCP
      port: 5432
  egress:
  - to: []  # DNS only
    ports:
    - protocol: UDP
      port: 53
```

### 10. Explain Service types and their use cases with examples.

**Answer:**

**ClusterIP (Default):**
- **Use Case:** Internal service communication
- **Characteristics:** Only accessible within cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

**NodePort:**
- **Use Case:** External access during development, simple load balancer setup
- **Characteristics:** Exposes service on each node's IP at a static port

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Optional: 30000-32767 range
```

**LoadBalancer:**
- **Use Case:** Production external access with cloud provider integration
- **Characteristics:** Provisions external load balancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:us-west-2:123456789:certificate/12345"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 443
    targetPort: 8080
    protocol: TCP
  loadBalancerSourceRanges:
  - "10.0.0.0/8"
  - "192.168.0.0/16"
```

**ExternalName:**
- **Use Case:** Service discovery for external services, database migration
- **Characteristics:** Maps to external DNS name

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: db.example.com
  ports:
  - port: 3306
```

**Headless Service:**
- **Use Case:** StatefulSets, custom load balancing, service discovery
- **Characteristics:** No cluster IP assigned, direct pod IP resolution

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

## Section 4: Storage

### 11. Explain Persistent Volume lifecycle and dynamic provisioning.

**Answer:**

**PV Lifecycle Phases:**

**1. Provisioning:**
- **Static:** Admin pre-creates PVs
- **Dynamic:** StorageClass automatically provisions PVs

**2. Binding:**
- PVC requests storage with specific criteria
- Control plane matches PVC to suitable PV
- One-to-one binding relationship

**3. Using:**
- Pod uses PVC as volume
- Cluster protects in-use PVs from deletion

**4. Reclaiming:**
- **Retain:** Manual cleanup required
- **Delete:** PV and external storage deleted
- **Recycle:** Deprecated, basic scrub

**Static Provisioning Example:**
```yaml
# Persistent Volume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /data/mysql

---
# Persistent Volume Claim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
```

**Dynamic Provisioning Setup:**
```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
  encrypted: "true"
  iops: "3000"
  throughput: "250"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer

---
# PVC using StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: fast-ssd
```

**Volume Expansion:**
```yaml
# Update PVC size
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi  # Increased from 50Gi
  storageClassName: fast-ssd
```

### 12. How does storage work with StatefulSets?

**Answer:**

**StatefulSet Storage Characteristics:**
- Each pod gets its own PVC
- Stable storage that persists across pod rescheduling
- VolumeClaimTemplates define storage requirements
- Storage scales with StatefulSet replicas

**StatefulSet with Storage Example:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: admin
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /data/db
        - name: config
          mountPath: /data/configdb
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1"
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
  - metadata:
      name: config
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 1Gi
```

**Scaling Behavior:**
```bash
# Scale up - new PVCs created automatically
kubectl scale statefulset mongodb --replicas=5

# Scale down - PVCs retained for data safety
kubectl scale statefulset mongodb --replicas=2
# PVCs mongodb-data-2, mongodb-data-3, mongodb-data-4 remain
```

**Persistent Identity:**
- Pod: mongodb-0, mongodb-1, mongodb-2
- PVCs: mongodb-data-0, mongodb-data-1, mongodb-data-2
- DNS: mongodb-0.mongodb.default.svc.cluster.local

**Recovery Scenario:**
```bash
# If mongodb-1 pod fails and gets rescheduled
# New pod mongodb-1 automatically gets the same PVC
# Data persists across pod recreation
```

## Section 5: Security

### 13. Design RBAC policies for a development team scenario.

**Answer:**

**Scenario:** Development team needs:
- Full access to their namespace "dev-team-a"
- Read-only access to shared "monitoring" namespace
- Ability to view cluster-wide resources (nodes, storage classes)
- Cannot access other team namespaces

**Implementation:**

**1. Namespace-specific Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team-a
  name: dev-team-full-access
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps", "extensions"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies", "ingresses"]
  verbs: ["*"]
- apiGroups: ["autoscaling"]
  resources: ["horizontalpodautoscalers"]
  verbs: ["*"]
```

**2. Monitoring Namespace Read Access:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: monitoring
  name: monitoring-read-only
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
```

**3. Cluster-wide Read Access:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["nodes", "namespaces", "persistentvolumes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
```

**4. Service Account:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-team-a-sa
  namespace: dev-team-a
```

**5. Role Bindings:**
```yaml
# Full access to team namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-a-binding
  namespace: dev-team-a
subjects:
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: bob@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: dev-team-a-sa
  namespace: dev-team-a
roleRef:
  kind: Role
  name: dev-team-full-access
  apiGroup: rbac.authorization.k8s.io

---
# Monitoring read access
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitoring-read-binding
  namespace: monitoring
subjects:
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: bob@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: monitoring-read-only
  apiGroup: rbac.authorization.k8s.io

---
# Cluster read access
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dev-team-a-cluster-read
subjects:
- kind: User
  name: alice@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: bob@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

**6. Testing RBAC:**
```bash
# Test as user alice
kubectl auth can-i create pods --namespace=dev-team-a --as=alice@company.com
# yes

kubectl auth can-i create pods --namespace=dev-team-b --as=alice@company.com
# no

kubectl auth can-i get nodes --as=alice@company.com
# yes

kubectl auth can-i delete nodes --as=alice@company.com
# no
```

### 14. Implement Pod Security Standards and Security Contexts.

**Answer:**

**Pod Security Standards Overview:**
- **Privileged:** Unrestricted policy (default)
- **Baseline:** Minimally restrictive, prevents known privilege escalations
- **Restricted:** Heavily restricted, follows security hardening best practices

**Namespace-level Pod Security:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Security Context Examples:**

**1. Restricted Security Context:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
  namespace: production
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 2000
    fsGroup: 3000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx:1.21
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      runAsUser: 1000
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
```

**2. Database with Security Context:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      securityContext:
        runAsUser: 999
        runAsGroup: 999
        fsGroup: 999
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: postgres
        image: postgres:14
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: false  # Postgres needs write access
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**3. PolicyEngine with OPA Gatekeeper:**
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: requiredsecuritycontext
spec:
  crd:
    spec:
      names:
        kind: RequiredSecurityContext
      validation:
        properties:
          runAsNonRoot:
            type: boolean
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package requiredsecuritycontext
        
        violation[{"msg": msg}] {
            input.review.object.kind == "Pod"
            not input.review.object.spec.securityContext.runAsNonRoot == true
            msg := "Containers must run as non-root user"
        }
        
        violation[{"msg": msg}] {
            input.review.object.kind == "Pod"
            container := input.review.object.spec.containers[_]
            container.securityContext.allowPrivilegeEscalation == true
            msg := "Privilege escalation is not allowed"
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequiredSecurityContext
metadata:
  name: must-run-as-nonroot
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    namespaces: ["production", "staging"]
  parameters:
    runAsNonRoot: true
```

### 15. Secrets management and external secret integration.

**Answer:**

**Native Kubernetes Secrets:**
```yaml
# Creating secret from literal values
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: production
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded "admin"
  password: cGFzc3dvcmQ=  # base64 encoded "password"
stringData:
  api-key: "super-secret-api-key"  # automatically base64 encoded

---
# Using secrets in pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: username
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: api-key
        volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: app-secrets
          defaultMode: 0400
```

**External Secrets Operator with AWS Secrets Manager:**
```yaml
# SecretStore configuration
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-west-2
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa

---
# ExternalSecret definition
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  refreshInterval: 30s
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: database-secret
    creationPolicy: Owner
    template:
      type: Opaque
      data:
        username: "{{ .username }}"
        password: "{{ .password }}"
        connection-string: "postgresql://{{ .username }}:{{ .password }}@{{ .host }}:5432/{{ .database }}"
  data:
  - secretKey: username
    remoteRef:
      key: prod/database
      property: username
  - secretKey: password
    remoteRef:
      key: prod/database
      property: password
  - secretKey: host
    remoteRef:
      key: prod/database
      property: host
  - secretKey: database
    remoteRef:
      key: prod/database
      property: database
```

**HashiCorp Vault Integration:**
```yaml
# Vault Agent Injector
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-vault
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "app-role"
        vault.hashicorp.com/agent-inject-secret-config: "secret/data/myapp"
        vault.hashicorp.com/agent-inject-template-config: |
          {{- with secret "secret/data/myapp" -}}
          export DATABASE_URL="{{ .Data.data.database_url }}"
          export API_KEY="{{ .Data.data.api_key }}"
          {{- end }}
    spec:
      serviceAccountName: app-service-account
      containers:
      - name: app
        image: myapp:latest
        command: ["/bin/sh"]
        args: ["-c", "source /vault/secrets/config && exec /app/start.sh"]
```

**Sealed Secrets (GitOps-friendly):**
```yaml
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Create SealedSecret
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mysecret
  namespace: production
spec:
  encryptedData:
    username: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEQAx...
    password: AyBQ2eEKWfpd3+DwQR7JQsf8ck4dJKD2A...
  template:
    metadata:
      name: mysecret
      namespace: production
    type: Opaque
```

**Best Practices:**
1. **Never store secrets in Git repositories**
2. **Use least privilege access for secret consumers**
3. **Rotate secrets regularly**
4. **Encrypt secrets at rest in etcd**
5. **Audit secret access**
6. **Use external secret management systems for production**

## Section 6: Monitoring & Troubleshooting

### 16. Design a complete observability stack for Kubernetes.

**Answer:**

**Observability Stack Components:**

**1. Metrics Collection (Prometheus):**
```yaml
# Prometheus configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 30s
      evaluation_interval: 30s
    
    rule_files:
      - "/etc/prometheus/rules/*.yml"
    
    scrape_configs:
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):10250'
        target_label: __address__
        replacement: '${1}:9100'
    
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
    
    - job_name: 'kube-state-metrics'
      static_configs:
      - targets: ['kube-state-metrics:8080']

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.40.0
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus/'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--storage.tsdb.retention.time=30d'
          - '--web.enable-lifecycle'
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config-volume
          mountPath: /etc/prometheus
        - name: storage-volume
          mountPath: /prometheus
      volumes:
      - name: config-volume
        configMap:
          name: prometheus-config
      - name: storage-volume
        persistentVolumeClaim:
          claimName: prometheus-storage
```

**2. Metrics Visualization (Grafana):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:9.0.0
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: grafana-secret
              key: admin-password
        - name: GF_INSTALL_PLUGINS
          value: "grafana-kubernetes-app"
        ports:
        - containerPort: 3000
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
        - name: grafana-config
          mountPath: /etc/grafana/provisioning
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-storage
      - name: grafana-config
        configMap:
          name: grafana-config

---
# Grafana DataSource Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-config
data:
  datasources.yaml: |
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus:9090
      access: proxy
      isDefault: true
    - name: Loki
      type: loki
      url: http://loki:3100
      access: proxy
```

**3. Log Aggregation (Loki + Promtail):**
```yaml
# Loki configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: loki-config
data:
  loki.yaml: |
    auth_enabled: false
    server:
      http_listen_port: 3100
    ingester:
      lifecycler:
        address: 127.0.0.1
        ring:
          kvstore:
            store: inmemory
          replication_factor: 1
        final_sleep: 0s
    schema_config:
      configs:
        - from: 2020-10-24
          store: boltdb-shipper
          object_store: filesystem
          schema: v11
          index:
            prefix: index_
            period: 24h
    storage_config:
      boltdb_shipper:
        active_index_directory: /loki/boltdb-shipper-active
        cache_location: /loki/boltdb-shipper-cache
        shared_store: filesystem
      filesystem:
        directory: /loki/chunks
    limits_config:
      enforce_metric_name: false
      reject_old_samples: true
      reject_old_samples_max_age: 168h

---
# Promtail DaemonSet for log collection
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: promtail
spec:
  selector:
    matchLabels:
      name: promtail
  template:
    metadata:
      labels:
        name: promtail
    spec:
      serviceAccountName: promtail
      containers:
      - name: promtail
        image: grafana/promtail:2.6.0
        args:
        - -config.file=/etc/promtail/config.yml
        volumeMounts:
        - name: config
          mountPath: /etc/promtail
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: promtail-config
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

**4. Distributed Tracing (Jaeger):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:1.35
        env:
        - name: COLLECTOR_ZIPKIN_HOST_PORT
          value: ":9411"
        - name: SPAN_STORAGE_TYPE
          value: "elasticsearch"
        - name: ES_SERVER_URLS
          value: "http://elasticsearch:9200"
        ports:
        - containerPort: 16686  # UI
        - containerPort: 14268  # HTTP collector
        - containerPort: 9411   # Zipkin
```

**5. Alerting Rules:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
data:
  alerts.yml: |
    groups:
    - name: kubernetes-alerts
      rules:
      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[5m]) * 60 * 5 > 0
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping
          description: Pod {{ $labels.namespace }}/{{ $labels.pod }} is restarting frequently
      
      - alert: NodeMemoryHigh
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.1
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: Node {{ $labels.instance }} memory usage is high
          description: Node {{ $labels.instance }} has less than 10% memory available
      
      - alert: PodMemoryHigh
        expr: (container_memory_working_set_bytes / container_spec_memory_limit_bytes) > 0.9
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: Pod {{ $labels.namespace }}/{{ $labels.pod }} memory usage is high
          description: Pod {{ $labels.namespace }}/{{ $labels.pod }} is using more than 90% of its memory limit
```

**6. Application Instrumentation Example:**
```yaml
# Application with observability
apiVersion: apps/v1
kind: Deployment
metadata:
  name: instrumented-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: instrumented-app
  template:
    metadata:
      labels:
        app: instrumented-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: app
        image: myapp:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: JAEGER_AGENT_HOST
          value: jaeger-agent
        - name: JAEGER_AGENT_PORT
          value: "6831"
        - name: LOG_LEVEL
          value: "info"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

### 17. Troubleshoot a high-latency application scenario.

**Answer:**

**Scenario:** Users reporting slow response times for the e-commerce API

**Step 1: Initial Assessment**
```bash
# Check overall cluster health
kubectl get nodes
kubectl top nodes
kubectl get pods --all-namespaces | grep -v Running

# Check application status
kubectl get pods -n ecommerce -l app=api
kubectl describe pods -n ecommerce -l app=api
```

**Step 2: Analyze Metrics**
```bash
# Check CPU and memory usage
kubectl top pods -n ecommerce
kubectl describe hpa -n ecommerce

# Review recent scaling events
kubectl get events -n ecommerce --sort-by='.lastTimestamp'
```

**Step 3: Examine Application Logs**
```bash
# Check application logs for errors
kubectl logs -n ecommerce -l app=api --tail=100 --since=10m

# Look for specific patterns
kubectl logs -n ecommerce -l app=api | grep -E "(ERROR|WARN|timeout|slow)"
```

**Step 4: Network Analysis**
```bash
# Check service endpoints
kubectl get endpoints -n ecommerce api-service

# Test internal connectivity
kubectl run debug --image=nicolaka/netshoot -it --rm -- /bin/bash
# Inside debug pod:
nslookup api-service.ecommerce.svc.cluster.local
curl -w "%{time_total}" http://api-service.ecommerce.svc.cluster.local/health
```

**Step 5: Database Performance Investigation**
```bash
# Check database pod resource usage
kubectl top pod -n ecommerce -l app=postgres

# Check database logs
kubectl logs -n ecommerce -l app=postgres --tail=50

# Check database connections
kubectl exec -n ecommerce postgres-0 -- psql -U admin -c "SELECT count(*) FROM pg_stat_activity;"
```

**Step 6: Deep Dive with Prometheus Queries**
```promql
# API response time percentiles
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{job="api"}[5m])) by (le))

# Error rate
sum(rate(http_requests_total{job="api",status=~"5.."}[5m])) / sum(rate(http_requests_total{job="api"}[5m])) * 100

# Database connection pool usage
database_connections_active / database_connections_max * 100

# Pod restart rate
rate(kube_pod_container_status_restarts_total{namespace="ecommerce"}[5m])
```

**Step 7: Identify Root Cause**

**Common Issues Found:**

**A. Resource Constraints:**
```yaml
# Current problematic configuration
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"

# Fixed configuration
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

**B. Database Connection Pool Exhaustion:**
```yaml
# Add connection pool configuration
env:
- name: DB_POOL_MAX_SIZE
  value: "20"
- name: DB_POOL_MIN_SIZE
  value: "5"
- name: DB_CONNECTION_TIMEOUT
  value: "30000"
```

**C. Missing Readiness Probe:**
```yaml
# Add proper readiness probe
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

**D. Inefficient Database Queries:**
```sql
-- Found slow query in logs
-- Before: Full table scan
SELECT * FROM orders WHERE customer_id = 123 ORDER BY created_at DESC;

-- After: Add index
CREATE INDEX idx_orders_customer_created ON orders(customer_id, created_at DESC);
```

**Step 8: Implement Fixes and Monitor**
```bash
# Apply resource updates
kubectl apply -f api-deployment-fixed.yaml

# Monitor the improvement
kubectl rollout status deployment/api -n ecommerce

# Watch metrics in real-time
watch kubectl top pods -n ecommerce
```

**Step 9: Prevent Future Issues**
```yaml
# Add comprehensive monitoring
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-monitor
spec:
  selector:
    matchLabels:
      app: api
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
# Add SLO-based alerting
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: api-slo-alerts
spec:
  groups:
  - name: api-slo
    rules:
    - alert: APIHighLatency
      expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{job="api"}[5m])) by (le)) > 0.5
      for: 2m
      annotations:
        summary: API 95th percentile latency is above 500ms
    
    - alert: APIHighErrorRate
      expr: sum(rate(http_requests_total{job="api",status=~"5.."}[5m])) / sum(rate(http_requests_total{job="api"}[5m])) > 0.01
      for: 2m
      annotations:
        summary: API error rate is above 1%
```

**Resolution Summary:**
- **Root Cause:** Insufficient CPU resources + database connection pool exhaustion
- **Fix:** Increased resource limits and optimized database configuration
- **Prevention:** Added proper monitoring, alerting, and load testing
- **Result:** P95 latency reduced from 2.5s to 180ms

## Section 8: Advanced Topics

### 18. Design and implement a Custom Resource Definition (CRD) with controller.

**Answer:**

**Scenario:** Create a custom backup resource that manages automated database backups

**1. CRD Definition:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databasebackups.backup.example.com
spec:
  group: backup.example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              database:
                type: object
                properties:
                  type:
                    type: string
                    enum: ["postgresql", "mysql", "mongodb"]
                  host:
                    type: string
                  port:
                    type: integer
                  name:
                    type: string
                  secretRef:
                    type: object
                    properties:
                      name:
                        type: string
                      namespace:
                        type: string
                required: ["type", "host", "name", "secretRef"]
              schedule:
                type: string
                pattern: '^(\*|([0-9]|1[0-9]|2[0-9]|3[0-9]|4[0-9]|5[0-9])|\*\/([0-9]|1[0-9]|2[0-9]|3[0-9]|4[0-9]|5[0-9])) (\*|([0-9]|1[0-9]|2[0-3])|\*\/([0-9]|1[0-9]|2[0-3])) (\*|([1-9]|1[0-9]|2[0-9]|3[0-1])|\*\/([1-9]|1[0-9]|2[0-9]|3[0-1])) (\*|([1-9]|1[0-2])|\*\/([1-9]|1[0-2])) (\*|([0-6])|\*\/([0-6]))$'
              retention:
                type: object
                properties:
                  keepDaily:
                    type: integer
                    minimum: 1
                  keepWeekly:
                    type: integer
                    minimum: 0
                  keepMonthly:
                    type: integer
                    minimum: 0
                required: ["keepDaily"]
              storage:
                type: object
                properties:
                  type:
                    type: string
                    enum: ["s3", "gcs", "pvc"]
                  config:
                    type: object
                    x-kubernetes-preserve-unknown-fields: true
                required: ["type", "config"]
            required: ["database", "schedule", "retention", "storage"]
          status:
            type: object
            properties:
              lastBackup:
                type: string
                format: date-time
              nextBackup:
                type: string
                format: date-time
              phase:
                type: string
                enum: ["Pending", "Running", "Succeeded", "Failed"]
              message:
                type: string
              backupHistory:
                type: array
                items:
                  type: object
                  properties:
                    timestamp:
                      type: string
                      format: date-time
                    status:
                      type: string
                    location:
                      type: string
                    size:
                      type: string
    additionalPrinterColumns:
    - name: Database
      type: string
      jsonPath: .spec.database.type
    - name: Schedule
      type: string
      jsonPath: .spec.schedule
    - name: Last Backup
      type: string
      jsonPath: .status.lastBackup
    - name: Status
      type: string
      jsonPath: .status.phase
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: databasebackups
    singular: databasebackup
    kind: DatabaseBackup
    shortNames:
    - dbbackup
```

**2. Custom Resource Example:**
```yaml
apiVersion: backup.example.com/v1
kind: DatabaseBackup
metadata:
  name: postgres-prod-backup
  namespace: production
spec:
  database:
    type: postgresql
    host: postgres.production.svc.cluster.local
    port: 5432
    name: ecommerce
    secretRef:
      name: postgres-credentials
      namespace: production
  schedule: "0 2 * * *"  # Daily at 2 AM
  retention:
    keepDaily: 7
    keepWeekly: 4
    keepMonthly: 12
  storage:
    type: s3
    config:
      bucket: company-database-backups
      region: us-west-2
      prefix: postgres-prod/
      storageClass: STANDARD_IA
```

**3. Controller Implementation (Go with controller-runtime):**
```go
package controllers

import (
    "context"
    "fmt"
    "time"
    
    batchv1 "k8s.io/api/batch/v1"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/log"
    
    backupv1 "github.com/company/backup-operator/api/v1"
)

type DatabaseBackupReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

func (r *DatabaseBackupReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    logger := log.FromContext(ctx)
    
    // Fetch the DatabaseBackup instance
    var dbBackup backupv1.DatabaseBackup
    if err := r.Get(ctx, req.NamespacedName, &dbBackup); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // Check if it's time for next backup
    nextBackup := r.calculateNextBackup(dbBackup.Spec.Schedule, dbBackup.Status.LastBackup)
    now := time.Now()
    
    if now.Before(nextBackup) {
        // Requeue when it's time for next backup
        return ctrl.Result{RequeueAfter: time.Until(nextBackup)}, nil
    }
    
    // Create backup job
    job, err := r.createBackupJob(ctx, &dbBackup)
    if err != nil {
        logger.Error(err, "Failed to create backup job")
        return ctrl.Result{}, err
    }
    
    // Update status
    dbBackup.Status.Phase = "Running"
    dbBackup.Status.NextBackup = r.calculateNextBackup(dbBackup.Spec.Schedule, &now)
    dbBackup.Status.Message = fmt.Sprintf("Backup job %s created", job.Name)
    
    if err := r.Status().Update(ctx, &dbBackup); err != nil {
        return ctrl.Result{}, err
    }
    
    return ctrl.Result{RequeueAfter: time.Minute * 5}, nil
}

func (r *DatabaseBackupReconciler) createBackupJob(ctx context.Context, dbBackup *backupv1.DatabaseBackup) (*batchv1.Job, error) {
    job := &batchv1.Job{
        ObjectMeta: metav1.ObjectMeta{
            Name:      fmt.Sprintf("%s-%d", dbBackup.Name, time.Now().Unix()),
            Namespace: dbBackup.Namespace,
            OwnerReferences: []metav1.OwnerReference{
                {
                    APIVersion: dbBackup.APIVersion,
                    Kind:       dbBackup.Kind,
                    Name:       dbBackup.Name,
                    UID:        dbBackup.UID,
                    Controller: &[]bool{true}[0],
                },
            },
        },
        Spec: batchv1.JobSpec{
            Template: corev1.PodTemplateSpec{
                Spec: corev1.PodSpec{
                    RestartPolicy: corev1.RestartPolicyNever,
                    Containers: []corev1.Container{
                        {
                            Name:  "backup",
                            Image: r.getBackupImage(dbBackup.Spec.Database.Type),
                            Env:   r.buildEnvVars(dbBackup),
                            Command: r.getBackupCommand(dbBackup),
                        },
                    },
                },
            },
        },
    }
    
    if err := r.Create(ctx, job); err != nil {
        return nil, err
    }
    
    return job, nil
}

func (r *DatabaseBackupReconciler) getBackupImage(dbType string) string {
    switch dbType {
    case "postgresql":
        return "postgres:14"
    case "mysql":
        return "mysql:8.0"
    case "mongodb":
        return "mongo:5.0"
    default:
        return "alpine:latest"
    }
}

func (r *DatabaseBackupReconciler) getBackupCommand(dbBackup *backupv1.DatabaseBackup) []string {
    switch dbBackup.Spec.Database.Type {
    case "postgresql":
        return []string{
            "/bin/bash", "-c",
            "pg_dump -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME | aws s3 cp - s3://$S3_BUCKET/$S3_PREFIX/backup-$(date +%Y%m%d-%H%M%S).sql",
        }
    case "mysql":
        return []string{
            "/bin/bash", "-c",
            "mysqldump -h $DB_HOST -P $DB_PORT -u $DB_USER -p$DB_PASSWORD $DB_NAME | aws s3 cp - s3://$S3_BUCKET/$S3_PREFIX/backup-$(date +%Y%m%d-%H%M%S).sql",
        }
    default:
        return []string{"/bin/true"}
    }
}

func (r *DatabaseBackupReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&backupv1.DatabaseBackup{}).
        Owns(&batchv1.Job{}).
        Complete(r)
}
```

**4. RBAC for Controller:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: backup-controller
rules:
- apiGroups: ["backup.example.com"]
  resources: ["databasebackups"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["backup.example.com"]
  resources: ["databasebackups/status"]
  verbs: ["get", "update", "patch"]
- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "patch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: backup-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: backup-controller
subjects:
- kind: ServiceAccount
  name: backup-controller
  namespace: backup-system
```

**5. Controller Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backup-controller
  namespace: backup-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backup-controller
  template:
    metadata:
      labels:
        app: backup-controller
    spec:
      serviceAccountName: backup-controller
      containers:
      - name: controller
        image: backup-operator:v1.0.0
        command:
        - /manager
        args:
        - --leader-elect
        env:
        - name: WATCH_NAMESPACE
          value: ""
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8081
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /readyz
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 10
        resources:
          limits:
            cpu: 500m
            memory: 128Mi
          requests:
            cpu: 10m
            memory: 64Mi
```

### 19. Implement multi-tenancy with proper isolation.

**Answer:**

**Multi-Tenancy Strategy:**

**1. Namespace-per-Tenant Approach:**
```yaml
# Tenant namespace template
apiVersion: v1
kind: Namespace
metadata:
  name: tenant-acme-corp
  labels:
    tenant: acme-corp
    billing-tier: premium
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/warn: baseline
  annotations:
    scheduler.alpha.kubernetes.io/node-selector: "tenant=acme-corp"
```

**2. Resource Quotas per Tenant:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-acme-corp-quota
  namespace: tenant-acme-corp
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"
    persistentvolumeclaims: "10"
    services: "20"
    secrets: "50"
    configmaps: "50"
    count/deployments.apps: "20"
    count/jobs.batch: "10"

---
apiVersion: v1
kind: LimitRange
metadata:
  name: tenant-acme-corp-limits
  namespace: tenant-acme-corp
spec:
  limits:
  - type: Container
    default:
      cpu: "1"
      memory: "1Gi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "4"
      memory: "8Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
  - type: Pod
    max:
      cpu: "8"
      memory: "16Gi"
  - type: PersistentVolumeClaim
    max:
      storage: "100Gi"
    min:
      storage: "1Gi"
```

**3. Network Isolation:**
```yaml
# Deny all traffic by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: tenant-acme-corp
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Allow internal tenant communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal
  namespace: tenant-acme-corp
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: acme-corp
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: acme-corp
  - to: []  # DNS
    ports:
    - protocol: UDP
      port: 53

---
# Allow access to shared services
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-shared-services
  namespace: tenant-acme-corp
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: shared-services
    ports:
    - protocol: TCP
      port: 443  # Shared API gateway
    - protocol: TCP
      port: 5432  # Shared database (if allowed)
```

**4. RBAC for Tenant Users:**
```yaml
# Tenant admin role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: tenant-acme-corp
  name: tenant-admin
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["apps", "extensions", "networking.k8s.io", "autoscaling"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["*"]
# Restrict access to ResourceQuota and LimitRange
- apiGroups: [""]
  resources: ["resourcequotas", "limitranges"]
  verbs: ["get", "list", "watch"]

---
# Tenant developer role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: tenant-acme-corp
  name: tenant-developer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets", "persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log", "pods/exec"]
  verbs: ["get", "list", "create"]

---
# Bind users to roles
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: tenant-acme-corp-admins
  namespace: tenant-acme-corp
subjects:
- kind: User
  name: admin@acme-corp.com
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: tenant-acme-corp-admin
  namespace: tenant-acme-corp
roleRef:
  kind: Role
  name: tenant-admin
  apiGroup: rbac.authorization.k8s.io
```

**5. Node Isolation (if required):**
```yaml
# Taint nodes for specific tenants
kubectl taint nodes node-1 node-2 node-3 tenant=acme-corp:NoSchedule

# Tenant workloads with tolerations
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tenant-app
  namespace: tenant-acme-corp
spec:
  template:
    spec:
      tolerations:
      - key: "tenant"
        operator: "Equal"
        value: "acme-corp"
        effect: "NoSchedule"
      nodeSelector:
        tenant: "acme-corp"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: tenant
                operator: In
                values: ["acme-corp"]
```

**6. Storage Isolation:**
```yaml
# Tenant-specific StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: tenant-acme-corp-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
  encrypted: "true"
  # Tenant-specific KMS key
  kmsKeyId: "arn:aws:kms:us-west-2:123456789:key/acme-corp-key"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: tenant
    values: ["acme-corp"]
```

**7. Monitoring Isolation:**
```yaml
# Tenant-specific ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: tenant-acme-corp-monitor
  namespace: tenant-acme-corp
spec:
  selector:
    matchLabels:
      tenant: acme-corp
  endpoints:
  - port: metrics
    interval: 30s

---
# Grafana dashboard access control
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-acme-corp
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "Acme Corp Tenant Dashboard",
        "templating": {
          "list": [
            {
              "name": "namespace",
              "query": "label_values(kube_pod_info{namespace=~\"tenant-acme-corp.*\"}, namespace)"
            }
          ]
        }
      }
    }
```

**8. Tenant Operator for Automation:**
```yaml
apiVersion: tenancy.example.com/v1
kind: Tenant
metadata:
  name: acme-corp
spec:
  displayName: "Acme Corporation"
  contact:
    email: "admin@acme-corp.com"
  quotas:
    cpu: "20"
    memory: "40Gi"
    storage: "1Ti"
    pods: 100
  networkPolicy:
    isolation: strict
    allowedEgress:
    - shared-services
  users:
  - email: "admin@acme-corp.com"
    role: admin
  - email: "dev1@acme-corp.com"
    role: developer
  - email: "dev2@acme-corp.com"
    role: developer
```

**Best Practices for Multi-tenancy:**
1. **Security:** Default deny network policies, Pod Security Standards
2. **Resource Management:** Quotas and limits at multiple levels
3. **Monitoring:** Separate dashboards and alerting per tenant
4. **Automation:** Use operators to manage tenant lifecycle
5. **Compliance:** Audit logging and access controls
6. **Cost Management:** Resource tagging and chargeback mechanisms

---

## Quick Reference Commands

```bash
# Debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl describe pod <pod-name> | grep -A 10 Events
kubectl logs <pod-name> --previous
kubectl exec -it <pod-name> -- /bin/bash

# Resource Management
kubectl top nodes
kubectl top pods --all-namespaces
kubectl get hpa --all-namespaces

# Networking
kubectl get endpoints
kubectl get networkpolicies --all-namespaces
kubectl port-forward <pod-name> 8080:80

# Security
kubectl auth can-i <verb> <resource> --as=<user>
kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide

# Storage
kubectl get pv,pvc --all-namespaces
kubectl describe storageclass

# Advanced
kubectl get crd
kubectl api-resources
kubectl explain <resource>
```
