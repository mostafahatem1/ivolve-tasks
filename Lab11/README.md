# Lab 11: Namespace Management and Resource Quota Enforcement

## Objective

In this lab, we will:

1. Create a Kubernetes namespace named `ivolve`.
2. Apply a ResourceQuota to the namespace.
3. Limit the maximum number of pods to `2`.
4. Verify that Kubernetes rejects the third pod.

## Prerequisites

The following tools are required:

- Docker
- Minikube
- kubectl

Check the installed tools:

```bash
docker --version
minikube version
kubectl version --client
```

## Step 1: Start the Minikube Cluster

Start a Minikube cluster using the Docker driver:

```bash
minikube start -p lab11 --driver=docker
```

Switch to the `lab11` context:

```bash
kubectl config use-context lab11
```

Verify the current context:

```bash
kubectl config current-context
```

Verify the cluster node:

```bash
kubectl get nodes
```

Expected output:

```text
NAME    STATUS   ROLES           AGE   VERSION
lab11   Ready    control-plane   ...   ...
```

## Step 2: Create the Namespace Manifest

Create a file named `namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ivolve
```

Apply the manifest:

```bash
kubectl apply -f namespace.yaml
```

Expected output:

```text
namespace/ivolve created
```

Verify the namespace:

```bash
kubectl get namespace ivolve
```

Expected output:

```text
NAME     STATUS   AGE
ivolve   Active   ...
```

## Step 3: Create the ResourceQuota Manifest

Create a file named `resource-quota.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-quota
  namespace: ivolve
spec:
  hard:
    pods: "2"
```

The quota limits the maximum number of pods inside the `ivolve` namespace to two pods.

Apply the ResourceQuota:

```bash
kubectl apply -f resource-quota.yaml
```

Expected output:

```text
resourcequota/pod-quota created
```

## Step 4: Verify the ResourceQuota

Display the ResourceQuota:

```bash
kubectl get resourcequota -n ivolve
```

Expected output:

```text
NAME        REQUEST     LIMIT   AGE
pod-quota   pods: 0/2           ...
```

Display detailed information:

```bash
kubectl describe resourcequota pod-quota -n ivolve
```

Expected output:

```text
Name:       pod-quota
Namespace:  ivolve

Resource    Used   Hard
--------    ----   ----
pods        0      2
```

## Step 5: Create the First Two Pods

Create the first pod:

```bash
kubectl run pod-1 --image=nginx:alpine -n ivolve
```

Create the second pod:

```bash
kubectl run pod-2 --image=nginx:alpine -n ivolve
```

Verify the pods:

```bash
kubectl get pods -n ivolve
```

Verify the quota usage:

```bash
kubectl get resourcequota -n ivolve
```

Expected result:

```text
NAME        REQUEST     LIMIT   AGE
pod-quota   pods: 2/2           ...
```

## Step 6: Test ResourceQuota Enforcement

Try to create a third pod:

```bash
kubectl run pod-3 --image=nginx:alpine -n ivolve
```

Kubernetes rejects the third pod because the quota allows only two pods.

Expected error:

```text
Error from server (Forbidden): pods "pod-3" is forbidden:
exceeded quota: pod-quota, requested: pods=1, used: pods=2, limited: pods=2
```

## Step 7: Final Verification

Run the following commands:

```bash
kubectl get namespace ivolve
kubectl get pods -n ivolve
kubectl get resourcequota -n ivolve
kubectl describe resourcequota pod-quota -n ivolve
```

Final result:

```text
Namespace: ivolve
Maximum pods: 2
Current pods: 2
Third pod: Rejected
```

## Project Structure

```text
Lab11/
├── namespace.yaml
├── resource-quota.yaml
└── README.md
```

## Cleanup

Delete the test pods:

```bash
kubectl delete pod pod-1 pod-2 -n ivolve
```

Delete the namespace and all resources inside it:

```bash
kubectl delete namespace ivolve
```

Stop the Minikube cluster:

```bash
minikube stop -p lab11
```

Delete the Minikube cluster:

```bash
minikube delete -p lab11
```

## Verification Checklist

- [x] Created the `ivolve` namespace.
- [x] Created the `pod-quota` ResourceQuota.
- [x] Limited the namespace to two pods.
- [x] Created the first pod successfully.
- [x] Created the second pod successfully.
- [x] Verified quota usage as `2/2`.
- [x] Verified that Kubernetes rejected the third pod.
