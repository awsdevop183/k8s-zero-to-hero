# Kubernetes Cluster Creation, Quorum, and Kops Deployment

This guide explains:

- Different ways to create a Kubernetes cluster
- What quorum means in Kubernetes
- The basic idea behind the RAFT algorithm
- How to deploy a Kubernetes cluster using **Kops**
- How to run quick smoke tests
- How to delete the cluster when done

---

## Ways to Create a Kubernetes Cluster

Kubernetes clusters can be created in multiple ways depending on your use case, environment, and scale.

### 1. Kops
Kops is a popular tool for creating and managing production-grade Kubernetes clusters, especially on AWS.

### 2. Rancher / RKE
Rancher and RKE simplify Kubernetes cluster setup and management, especially for teams that want a UI-driven or enterprise-friendly experience.

### 3. Kubeadm
Kubeadm is the **official Kubernetes tool** for bootstrapping clusters.

### 4. K3s
K3s is a lightweight Kubernetes distribution that is great for:
- Low-resource environments
- Edge devices
- Labs and small setups

### 5. Minikube / Kind
These are commonly used for **local development** and learning:
- **Minikube**: Runs a local single-node or multi-node cluster
- **Kind**: Runs Kubernetes clusters using Docker containers

### 6. EKS / AKS / GKE
Managed Kubernetes services provided by cloud platforms:
- **EKS**: Amazon Elastic Kubernetes Service
- **AKS**: Azure Kubernetes Service
- **GKE**: Google Kubernetes Engine

### 7. Terraform (Infrastructure as Code)
Terraform can be used to provision infrastructure and automate Kubernetes cluster creation in a repeatable, version-controlled way.

---

## What is Quorum?

In Kubernetes, especially in the control plane and distributed systems, **quorum** means the minimum number of nodes that must agree for the system to continue making decisions safely.

Quorum is important because it prevents split-brain scenarios and keeps the cluster state consistent.

---

## RAFT Algorithm

RAFT is a consensus algorithm used in distributed systems to ensure multiple nodes agree on the same data and state.

### Formula

```text
(N - 1) / 2
```

This gives the **maximum number of node failures** the cluster can tolerate while still maintaining quorum.

### Example with 3 Nodes

```text
N = 3

(3 - 1) / 2
= 2 / 2
= 1
```

This means a 3-node setup can tolerate **1 node failure**.

---

## Why Do We Need Quorum?

Quorum is required to ensure the cluster can safely process updates and maintain consistency.

If quorum is lost, for example if too many nodes go down, the cluster may become:

- Read-only
- Unavailable for writes
- Unable to elect a leader
- Unable to safely update cluster state

In short, **without quorum, the cluster cannot safely function for write operations**.

---

## Deploy a Cluster with Kops

You can deploy a cluster using either:

- An EC2 instance
- A local machine

### Required Tools

Make sure the following are installed:

1. Kops
2. Kubectl
3. AWS CLI

---

## Set the Kops State Store

```bash
export KOPS_STATE_STORE=s3://realreview.com
```

To make it persistent, add it to your `~/.bashrc`:

```bash
echo 'export KOPS_STATE_STORE=s3://realreview.com' >> ~/.bashrc
source ~/.bashrc
```

---

## Enable Bash Autocompletion

```bash
# Install the package
sudo apt-get install bash-completion -y

# Source the main completion script
source /usr/share/bash-completion/bash_completion

# Add kubectl completion to .bashrc
echo 'source <(kubectl completion bash)' >> ~/.bashrc

# Optional: alias kubectl as k
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc

# Reload bashrc
source ~/.bashrc
```

---

## Create a Kubernetes Cluster with Kops

```bash
kops create cluster k8s.aws365.shop \
  --state=s3://realreview.com \
  --cloud=aws \
  --zones=us-east-1a \
  --control-plane-count=1 \
  --control-plane-zones=us-east-1a \
  --control-plane-size=t3.medium \
  --node-count=2 \
  --node-size=t3.medium \
  --dns-zone=aws365.shop \
  --networking=calico \
  --topology=public \
  --api-loadbalancer-type=public
```

### Apply the Cluster Configuration

After creating the cluster definition, apply it:

```bash
kops update cluster k8s.aws365.shop --yes
```

### Validate the Cluster

```bash
kops validate cluster --wait 10m
```

---

## Smoke Testing

Run the following commands to verify the cluster is working properly:

```bash
kubectl get nodes
kubectl get ns
kubectl get po -n kube-system
```

### Deploy a Test Application

```bash
kubectl create deployment test --image=nginx --replicas=3
kubectl expose deployment test --port=80 --type=LoadBalancer
```

### Clean Up the Test Service

```bash
kubectl delete svc test
```

You can also remove the deployment if needed:

```bash
kubectl delete deployment test
```

---

## Delete the Cluster

When you are done, delete the cluster:

```bash
kops delete cluster k8s.aws365.shop --yes
```

---

## Notes

- For production environments, it is better to use **multiple control plane nodes** to maintain quorum and high availability.
- A single control plane node is useful for learning and demos, but it is **not highly available**.
- Always monitor cloud resource usage to avoid unexpected AWS charges.

---

## Summary

This README covered:

- Multiple ways to create a Kubernetes cluster
- Quorum and why it matters
- RAFT algorithm basics
- Kops-based cluster deployment
- Smoke testing
- Cluster cleanup

Use this as a quick reference while practicing Kubernetes cluster setup.
