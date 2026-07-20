# Lab 10: Node Isolation Using Taints in Kubernetes

## Overview

This lab demonstrates how to isolate a Kubernetes node using a taint.

The lab includes:

- Running a Minikube cluster with 2 nodes.
- Applying the taint `node=worker:NoSchedule` to one node.
- Verifying the taint using `kubectl describe`.

## Requirements

Make sure the following tools are installed:

- Docker
- Minikube
- kubectl

Verify them with:

```bash
docker --version
minikube version
kubectl version --client
```

## 1. Create a Two-Node Minikube Cluster

Create a new Minikube profile named `lab10`:

```bash
minikube start -p lab10 --nodes 2 --driver=docker
```

Check the cluster status:

```bash
minikube status -p lab10
```

Switch to the correct Kubernetes context:

```bash
kubectl config use-context lab10
```

Verify the current context:

```bash
kubectl config current-context
```

Expected output:

```text
lab10
```

## 2. Verify the Nodes

List all nodes:

```bash
kubectl get nodes
```

Example output:

```text
NAME        STATUS   ROLES           AGE   VERSION
lab10       Ready    control-plane   2m    v1.xx.x
lab10-m02   Ready    <none>          1m    v1.xx.x
```

The first node is the control-plane node, and the second node is the worker node.

## 3. Get the Worker Node Name

Store the worker node name in a variable:

```bash
WORKER_NODE=$(kubectl get nodes --no-headers | awk '$3 !~ /control-plane/ {print $1; exit}')
```

Verify the value:

```bash
echo "$WORKER_NODE"
```

Example output:

```text
lab10-m02
```

## 4. Apply the Taint

Apply the required taint to the worker node:

```bash
kubectl taint nodes "$WORKER_NODE" node=worker:NoSchedule
```

Expected output:

```text
node/lab10-m02 tainted
```

The taint contains:

```text
Key: node
Value: worker
Effect: NoSchedule
```

The `NoSchedule` effect prevents new pods from being scheduled on this node unless they have a matching toleration.

## 5. Verify the Taint

Describe all nodes:

```bash
kubectl describe nodes
```

In the worker node details, the following line should appear:

```text
Taints: node=worker:NoSchedule
```

To display only node names and taints:

```bash
kubectl describe nodes | grep -E "^Name:|^Taints:"
```

Example output:

```text
Name:               lab10
Taints:             <none>
Name:               lab10-m02
Taints:             node=worker:NoSchedule
```

You can also verify the taint directly using:

```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

## All Commands

```bash
minikube start -p lab10 --nodes 2 --driver=docker

kubectl config use-context lab10
kubectl get nodes

WORKER_NODE=$(kubectl get nodes --no-headers | awk '$3 !~ /control-plane/ {print $1; exit}')
echo "$WORKER_NODE"

kubectl taint nodes "$WORKER_NODE" node=worker:NoSchedule

kubectl describe nodes
kubectl describe nodes | grep -E "^Name:|^Taints:"
```

## Remove the Taint

To remove the taint later:

```bash
kubectl taint nodes "$WORKER_NODE" node=worker:NoSchedule-
```

Verify that it was removed:

```bash
kubectl describe node "$WORKER_NODE" | grep Taints
```

Expected output:

```text
Taints: <none>
```

## Stop the Cluster

Stop the Minikube cluster without deleting it:

```bash
minikube stop -p lab10
```

## Delete the Cluster

Delete the cluster completely:

```bash
minikube delete -p lab10
```

## Lab Verification Checklist

- [x] Minikube cluster created with 2 nodes.
- [x] Both nodes are in `Ready` status.
- [x] Worker node selected.
- [x] Taint `node=worker:NoSchedule` applied.
- [x] Taint verified using `kubectl describe nodes`.
