# PART 1 - Kubernetes Cluster Setup using Kubeadm

## 1. Create Nodes

- 1 Master Node, 2 Worker Nodes

**Reason:** Defines the cluster layout for control and workload distribution.

## 2. Set Hostnames

```bash
sudo hostnamectl set-hostname master
sudo hostnamectl set-hostname worker1
sudo hostnamectl set-hostname worker2
```

**Reason:** Kubernetes uses hostnames to identify and manage nodes properly.

## 3. Update `/etc/hosts`

```bash
192.168.1.10 master
192.168.1.11 worker1
192.168.1.12 worker2
```

**Reason:** Ensures internal communication between nodes using hostnames.

## 4. Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

**Reason:** Kubernetes requires swap disabled for proper resource scheduling.

## 5. Install Containerd

```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

**Reason:** Container runtime is required to run containers on all nodes.

## 6. Install Kubernetes Tools

```bash
sudo apt install -y apt-transport-https curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

**Reason:** Tools to initialize, manage, and operate the Kubernetes cluster.

## 7. Initialize Control Plane (Master Node)

```bash
sudo kubeadm init --control-plane-endpoint=master --pod-network-cidr=192.168.0.0/16
```

**Reason:** Sets up the Kubernetes master node and cluster control plane.

## 8. Configure kubectl (Master Node)

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Reason:** Allows the master to manage the cluster using kubectl.

## 9. Join Worker Nodes

```bash
sudo kubeadm join master:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

**Reason:** Adds worker nodes to the cluster to run workloads.

## 10. Install Network Plugin (Calico)

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

**Reason:** Enables pod-to-pod networking across all nodes in the cluster.

## 11. Check Cluster Nodes

```bash
kubectl get nodes
```

**Reason:** Verifies all nodes are successfully joined and ready.

## 12. Verify System Pods

```bash
kubectl get pods -n kube-system
```

**Reason:** Confirms all control plane components are running correctly.


# PART 2 - Deploy a Sample Three-Tier Web Application

# Kubernetes Guestbook Application

## 1. Redis Leader

Create `redis-leader.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-leader
spec:
  ports:
    - port: 6379
  selector:
    app: redis
    role: leader
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-leader
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: leader
  template:
    metadata:
      labels:
        app: redis
        role: leader
    spec:
      containers:
      - name: redis
        image: redis:6.0
        ports:
        - containerPort: 6379
```

Deploy:

```bash
kubectl apply -f redis-leader.yaml
```

## 2. Redis Followers

Create `redis-follower.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
spec:
  ports:
    - port: 6379
  selector:
    app: redis
    role: follower
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-follower
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: follower
  template:
    metadata:
      labels:
        app: redis
        role: follower
    spec:
      containers:
      - name: redis
        image: redis:6.0
        ports:
        - containerPort: 6379
```

Deploy:

```bash
kubectl apply -f redis-follower.yaml
```

## 3. Frontend Application

Create `frontend.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
  selector:
    app: guestbook
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - name: php-guestbook
        image: gcr.io/google-samples/gb-frontend:v4
        ports:
        - containerPort: 80
        env:
        - name: GET_HOSTS_FROM
          value: "env"
```

Deploy:

```bash
kubectl apply -f frontend.yaml
```

## 4. Access Frontend

Get Node IP:

```bash
kubectl get nodes -o wide
```

Open browser:

```
http://<NodeIP>:30080
```

## 5. Test Guestbook

- Add entries on web UI
- Verify entries are stored in Redis

## 6. Verify Pods & Services

```bash
kubectl get pods
kubectl get svc
```
