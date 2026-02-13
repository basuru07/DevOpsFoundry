# Introduction to Kubernetes

---

## Outline

- What is Kubernetes?
- Why Kubernetes?
- Why learn Kubernetes?
- Kubernetes Architecture
- Components in Kubernetes
- Demo

---

## What is Kubernetes (K8s)?

Kubernetes comes from the Greek word meaning "Helmsman of a ship" or "pilot."

Kubernetes is a powerful open-source container orchestration tool originally developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF). It automates the deployment, scaling, and management of containerized applications.

- Built on Google's Borg system
- CNCF graduated project
- Initial stable release: v1.0 (July 21, 2015)
- Latest release as of May 16, 2025: v1.33.1

### Project Statistics

- Over 115,000 stars on GitHub
- 3,800+ contributors to K8s core
- 40,000+ forks

---

## Why Container Orchestration?

Container orchestration automates deployment, management, and scaling of containerized applications.

### Alternatives

- Apache Mesos
- Docker Swarm
- Others

---

## What Kubernetes Brings

- Native load balancing for all services
- Autoscale workloads
- Self-healing capabilities
- Manage both stateless and stateful applications
- Easy integration and support for 3rd party apps
- Seamless upgrading and rollback features (Blue-Green, Canary deployments)

---

## Key Concepts

### Pods

- Atomic unit or smallest "unit of work" in Kubernetes
- Can contain one or more containers sharing volumes, network namespace, and context
- Ephemeral (temporary)

### Services

- Unified method of accessing exposed workloads from Pods
- Durable resource (unlike Pods)
- Provides static cluster IP and static namespaced DNS name
- Not ephemeral

---

## Kubernetes Architecture Overview

### Control Plane Components

**Kube-apiserver**
- Provides forward-facing REST interface to the control plane
- All clients interact with Kubernetes strictly through the API Server
- Acts as gatekeeper handling authentication, authorization, validation, and admission control

**etcd**
- Cluster datastore (key-value store)
- Persists cluster state and configuration information
- Uses Raft Consensus for fault tolerance and consistency

**Kube-controller-manager**
- Primary daemon managing core component control loops
- Monitors cluster state via API server and steers cluster toward desired state
- Includes: Node controller, Job controller, EndpointSlice controller, ServiceAccount controller

**Kube-scheduler**
- Watches for newly created Pods with no assigned node
- Selects appropriate node based on resource requirements, constraints, affinity, data locality, and deadlines

### Worker Plane Components

**Kubelet**
- Node agent responsible for managing Pod lifecycle
- Understands YAML container manifests from various sources (file path, HTTP endpoint, etcd watch, HTTP server)

**Kube-proxy**
- Manages network rules on each node
- Performs connection forwarding and load balancing for cluster services
- Available proxy modes: Userspace, iptables, ipvs (default if supported)

**Container Runtime Engine**
- CRI (Container Runtime Interface) compatible application
- Options: Containerd (docker), Cri-o, Rkt, Kata, Virtlet

### Optional Services

**Cloud-controller-manager**
- Provides cloud-provider specific knowledge and integration
- Handles Node, Route, Service controllers and PersistentVolume Labels

---

## Core Objects

### Namespaces

Logical clusters or environments; primary method of partitioning a cluster or scoping access.

**Key Points:**
- Isolation of workloads, users, and resources
- Most K8s resources are namespaced
- Enable resource quotas and limits per namespace
- RBAC applied at namespace level for secure multi-tenancy

**Default Namespaces:**
- `default`: Default namespace for objects without explicit namespace
- `kube-system`: Home for Kubernetes-created objects and resources
- `kube-public`: Readable by all users; reserved for cluster bootstrapping

### Pods

Atomic unit or smallest unit of work. One or more containers sharing volumes, network namespace, and context.

**Pod Lifecycle Phase:** High-level summary of where the Pod is in its lifecycle.

### Labels

Key-value pairs identifying, describing, and grouping related sets of objects or resources.

- Not unique by themselves
- Have strict syntax with limited character set

### Selectors

Use labels to filter or select objects throughout Kubernetes.

### Services

Unified method of accessing exposed workloads from Pods.

- Durable resource with static cluster-unique IP
- Static namespaced DNS name format: `<service-name>.<namespace>.svc.cluster.local`
- Target Pods using equality-based selectors (=, ==, !=)
- kube-proxy provides simple load-balancing

#### Service Types

**ClusterIP**
- Exposes service on strictly cluster-internal virtual IP

**NodePort**
- Extends ClusterIP service
- Exposes port on every node's IP
- Port range: 30000-32767 (or statically defined)
- Access: `http://anyNodeIP:NodePort`

**LoadBalancer**
- Extends NodePort
- Works with external system to map cluster external IP to exposed service
- Access: `http://loadbalancerIP`

**ExternalName**
- Maps service to external DNS name

---

## Kubernetes Workloads

Workloads are higher-level objects managing Pods or other higher-level objects. All include a Pod Template as the base tier of management.

### ReplicaSet

Ensures specified number of Pod replicas are running.

- `replicas`: Desired number of Pod instances
- `selector`: Label selector for Pods managed by ReplicaSet

### Deployment

Declarative method of managing Pods via ReplicaSets.

- Provides rollback functionality and update control
- Updates managed through pod-template-hash label
- Each iteration creates unique label assigned to ReplicaSet and Pods

### DaemonSet

Ensures all nodes matching certain criteria run instance of supplied Pod.

- Bypass default scheduling mechanisms
- Ideal for cluster-wide services (log forwarding, health monitoring)
- Revisions managed via controller-revision-hash label

### StatefulSet

Tailored to managing Pods maintaining state.

- Pod identity (hostname, network, storage) persisted
- Assigned unique ordinal name: `<statefulset-name>-<ordinal-index>`

### Job

Job controller ensures one or more Pods execute and successfully terminate.

- Continues trying until satisfying completion and/or parallelism condition
- Pods not cleaned up until job itself is deleted

### CronJob

Extension of Job Controller providing cron-like schedule execution.

- Use cases: DB data ingestions, backup scripts

---

## Kubernetes Storage

### Volumes

Storage tied to Pod's lifecycle.

- Pod can have one or more volume types
- Consumed by any containers within Pod
- Survive Pod restarts (durability depends on volume type)

**Configuration:**
- `volumes`: List of volume objects attached to Pod (unique names)
- `volumeMounts`: Container-specific list referencing Pod volumes by name with mountPath

### Persistent Volumes (PV)

Represents storage resource at cluster level.

- Cluster-wide resource linked to backing storage provider (NFS, GCEPersistentDisk, RBD, etc.)
- Generally provisioned by administrator
- Lifecycle handled independently from Pods
- Cannot attach directly to Pod; relies on PersistentVolumeClaim

**Configuration:**
- `capacity.storage`: Total available storage
- `volumeMode`: Filesystem or Block
- `accessModes`: ReadWriteOnce, ReadOnlyMany, ReadWriteMany
- `persistentVolumeReclaimPolicy`: Retain or Delete
- `storageClassName`: Optional storage class name
- `mountOptions`: Optional mount options

### Persistent Volume Claims (PVC)

Namespaced request for storage.

- Satisfies set of requirements instead of mapping directly
- Ensures application's storage claim is portable across backends/providers

**Configuration:**
- `accessModes`: Subset of target PV/Storage Class modes
- `resources.requests.storage`: Desired storage amount
- `storageClassName`: Desired Storage Class name

**PV Phases:** Available, Bound, Released, Failed, Pending

### StorageClass

Abstraction on top of external storage resource (PV).

- Works with external storage system for dynamic provisioning
- Eliminates need for cluster admin to pre-provision PV

**Configuration:**
- `provisioner`: Driver for provisioning external storage
- `parameters`: Configuration parameters for provisioner
- `reclaimPolicy`: Retain or Delete

---

## Kubernetes Configurations

### ConfigMap

Externalized data stored within Kubernetes.

- Reference through environment variables, command-line arguments, or volume mounts
- Created from manifest, literals, directories, or files
- `data`: Contains key-value pairs of ConfigMap contents

### Secret

Functionally identical to ConfigMap but with security enhancements.

- Stored as base64 encoded content
- Encrypted at rest within etcd (if configured)
- Ideal for usernames, passwords, certificates, sensitive information

**Secret Types:**
- `docker-registry`: Container registry credentials
- `generic/Opaque`: Literal values from different sources
- `tls`: Certificate-based secret

**Configuration:**
- `type`: Type of secret
- `data`: Key-value pairs of base64 encoded content

---
