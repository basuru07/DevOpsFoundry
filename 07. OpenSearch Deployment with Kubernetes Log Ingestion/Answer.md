# OpenSearch Deployment with Kubernetes Log Ingestion

## Environment Details

| Component            | Details                     |
| -------------------- | --------------------------- |
| Operating System     | Ubuntu 24.04 LTS            |
| OpenSearch Version   | 2.19.1                      |
| OpenSearch Dashboard | 2.19.1                      |
| Kubernetes           | Existing Kubernetes Cluster |
| Log Collector        | Fluent Bit                  |
| Application          | Nginx Server                |

---

# Part 1: Deploy OpenSearch on Ubuntu VM

## 1. Update System Packages

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 2. Install Java

OpenSearch requires Java.

Check Java:

```bash
java -version
```

Install Java if required:

```bash
sudo apt install openjdk-21-jdk -y
```

---

## 3. Download OpenSearch

Download OpenSearch package:

```bash
wget https://artifacts.opensearch.org/releases/bundle/opensearch/2.19.1/opensearch-2.19.1-linux-x64.tar.gz
```

Extract:

```bash
tar -xzf opensearch-2.19.1-linux-x64.tar.gz
```

Move installation directory:

```bash
sudo mv opensearch-2.19.1 /opt/opensearch
```

---

## 4. Configure OpenSearch

Edit configuration:

```bash
sudo nano /opt/opensearch/config/opensearch.yml
```

Add the following configuration:

```yaml
cluster.name: opensearch-cluster

node.name: node-1

network.host: 0.0.0.0

http.port: 9200

discovery.type: single-node

plugins.security.disabled: true
```

---

## 5. Configure JVM Memory

Edit JVM configuration:

```bash
sudo nano /opt/opensearch/config/jvm.options
```

Set heap size:

```
-Xms1g
-Xmx1g
```

---

## 6. Start OpenSearch

Navigate to OpenSearch directory:

```bash
cd /opt/opensearch
```

Start OpenSearch:

```bash
nohup ./bin/opensearch > opensearch.log 2>&1 &
```

---

## 7. Verify OpenSearch Service

Check running process:

```bash
ps -ef | grep opensearch
```

Check port:

```bash
sudo ss -tulpn | grep 9200
```

Test API:

```bash
curl http://localhost:9200
```

Expected output:

```json
{
  "name": "node-1",
  "cluster_name": "opensearch-cluster",
  "version": {
    "number": "2.19.1"
  }
}
```

---

# Part 2: Install OpenSearch Dashboard

## 1. Download Dashboard

```bash
wget https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/2.19.1/opensearch-dashboards-2.19.1-linux-x64.tar.gz
```

Extract:

```bash
tar -xzf opensearch-dashboards-2.19.1-linux-x64.tar.gz
```

Move:

```bash
sudo mv opensearch-dashboards-2.19.1 /opt/opensearch-dashboards
```

---

## 2. Configure Dashboard

Edit:

```bash
sudo nano /opt/opensearch-dashboards/config/opensearch_dashboards.yml
```

Configuration:

```yaml
server.host: "0.0.0.0"

opensearch.hosts:
  - "http://localhost:9200"

opensearch.ssl.verificationMode: none
```

---

## 3. Start Dashboard

```bash
cd /opt/opensearch-dashboards

nohup ./bin/opensearch-dashboards > dashboard.log 2>&1 &
```

Verify:

```bash
sudo ss -tulpn | grep 5601
```

Access:

```
http://<VM-IP>:5601
```

---

# Part 3: Install Fluent Bit in Kubernetes

## 1. Add Fluent Bit Helm Repository

```bash
helm repo add fluent https://fluent.github.io/helm-charts

helm repo update
```

---

## 2. Install Fluent Bit

```bash
helm install fluent-bit fluent/fluent-bit
```

Verify:

```bash
kubectl get pods
```

---

# Fluent Bit Configuration

Create ConfigMap:

`fluent-bit-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config

data:

  fluent-bit.conf: |

    [SERVICE]
        Flush 1
        Log_Level info


    [INPUT]
        Name tail
        Path /var/log/containers/*.log
        Parser docker
        Tag kube.*


    [FILTER]
        Name kubernetes
        Match kube.*
        Merge_Log On


    [OUTPUT]
        Name opensearch
        Match *
        Host <OPENSEARCH_VM_IP>
        Port 9200
        Index kubernetes-logs
        Suppress_Type_Name On
```

Apply:

```bash
kubectl apply -f fluent-bit-config.yaml
```

Restart Fluent Bit:

```bash
kubectl rollout restart daemonset fluent-bit
```

---

# Part 4: Deploy Nginx in Kubernetes

## Nginx Deployment

Create:

`nginx.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 1

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

        image: nginx

        ports:

        - containerPort: 80
```

Deploy:

```bash
kubectl apply -f nginx.yaml
```

---

# Nginx Custom Log Format

Configure Nginx logging:

```nginx
log_format custom 
'$remote_addr - $remote_user [$time_local] '
'"$request" $status $body_bytes_sent '
'"$http_referer" "$http_user_agent" '
'"$http_x_forwarded_for"';


access_log /var/log/nginx/access.log custom;
```

This captures:

* Client IP
* HTTP request
* Response status
* User agent
* X-Forwarded-For IP

---

# Fluent Bit Nginx Parser

Add parser:

```ini
[PARSER]

Name nginx

Format regex

Regex ^(?<remote_addr>[^ ]*) - (?<remote_user>[^ ]*) \[(?<time_local>[^\]]*)\] "(?<request>[^"]*)" (?<status>[0-9]*) (?<body_bytes_sent>[0-9]*) "(?<http_referer>[^"]*)" "(?<http_user_agent>[^"]*)" "(?<http_x_forwarded_for>[^"]*)"
```

---

# Fluent Bit Filter

```ini
[FILTER]

Name parser

Match *

Key_Name log

Parser nginx
```

---

# Fluent Bit OpenSearch Output

```ini
[OUTPUT]

Name opensearch

Match *

Host <OPENSEARCH_VM_IP>

Port 9200

Index nginx-logs
```

---

# Generate Nginx Traffic

Download traffic script:

```bash
chmod +x generate-log-traffic.sh
```

Run:

```bash
./generate-log-traffic.sh
```

---

# Part 5: OpenSearch Dashboard Visualizations

## 1. Requests by Status Code

Type:

```
Pie Chart
```

Configuration:

```
Metric:
Count

Bucket:
Terms

Field:
status
```

Purpose:

Shows HTTP response distribution:

* 200 Success
* 403 Forbidden
* 404 Not Found

---

# 2. Top 10 Requesting IPs

Type:

```
Data Table
```

Field:

```
http_x_forwarded_for
```

Purpose:

Identify clients generating the most traffic.

---

# 3. Request Method Distribution

Type:

```
Bar Chart
```

Fields:

```
X-axis:
request_method

Y-axis:
Count
```

Purpose:

Shows usage of:

* GET
* POST
* PUT
* DELETE

---

# 4. User-Agent Distribution

Type:

```
Bar Chart / Data Table
```

Field:

```
http_user_agent
```

Purpose:

Analyze client types:

* Browsers
* Bots
* API tools

---

