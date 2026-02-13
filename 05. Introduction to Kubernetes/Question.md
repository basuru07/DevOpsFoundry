# 05. Kubernetes Assignment: Building a Kubernetes Cluster and Deploying a Three-Tier Web Application 

## Assignment Details

## Overview

- The goal of this assignment is to provide hands-on experience in deploying and managing Kubernetes infrastructure and workloads.  

- In the first part, you will create a Kubernetes cluster using kubeadm. This will help you understand the control plane architecture. 

- In the second part, you will use that cluster to deploy a practical, multi-tier web application. This step will expose you to core Kubernetes concepts such as Deployments, Services and Pods. By completing this assignment, you will become familiar with the complete lifecycle of a Kubernetes-powered application deployment.

## Assignment Steps

### PART 1. Set Up a Kubernetes Cluster using Kubeadm 

## Requirements

| Component | Specification |
|-----------|---------------|
| **Virtual Machines** | 3 VirtualBox VMs (1 Master, 2 Workers) |
| **Container Runtime** | containerd |
| **Networking** | CNI Plugin (Calico or Cilium recommended) |
| **Orchestration Tool** | kubeadm |

## Tasks

### 1. Provision Infrastructure

Set up the required virtual machines.

### 2. Initialize the First Control Plane Node

Use kubeadm init with the --control-plane-endpoint flag pointing to the master node. 

### 3. Join Worker Nodes

Use the kubeadm join command to add worker nodes to the cluster. 

### 4. Deploy Network Plugin

Install a CNI plugin to manage pod networking.

### 5. Validate Cluster Health

Ensure all nodes are in the Ready state. Verify  that the control plane components are functioning correctly.

## Deliverables

- A detailed report documenting each step taken, including configurations and commands used.
- Screenshots or outputs demonstrating the successful setup and health of the cluster. 




### PART 2. Deploy a Sample Three-Tier Web Application

## Sample Application 

For this assignment, use the PHP Guestbook application with Redis provided by the Kubernetes documentation. This application demonstrates a simple multi-tier architecture and is suitable for 
beginners. 

Components: - Frontend: PHP-based web interface - Backend: Redis leader and follower instances.

## Tasks

### 1. Deploy Redis Leader

Apply the manifest to deploy the Redis leader pod and service.

### 2. Deploy Redis Followers

Apply the manifest to deploy Redis follower pods and services.

### 3. Deploy Frontend Application

Apply the manifest to deploy the PHP frontend pods and service. 

### 4. Expose Frontend Service

Use a NodePort  service to expose the frontend application externally. 

(Since the LoadBalancer service type is not supported by default in a bare-metal environment, be sure to use a NodePort service type)

### 5. Access the Application

Retrieve the external IP and the  port to access the application through a web browser. 

### 6. Test Functionality

Add entries through the frontend and verify that they are stored and retrieved correctly via Redis. 

## Deliverables

- YAML manifests used for deployments and services. 
- Screenshots or outputs showing: 
          Running pods and services. 
          Accessing the application through a browser. 
          Successful addition and retrieval of guestbook entries.



