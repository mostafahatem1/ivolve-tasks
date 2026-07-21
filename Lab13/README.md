# Lab 13: Persistent Storage Setup for Application Logging

## Objective

This lab demonstrates how to configure persistent storage in Kubernetes for application logs using:

- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)
- `hostPath` storage
- `ReadWriteMany` access mode
- `Retain` reclaim policy

## Requirements

The PersistentVolume must have:

- Capacity: `1Gi`
- Storage type: `hostPath`
- Node path: `/mnt/app-logs`
- Access mode: `ReadWriteMany`
- Reclaim policy: `Retain`

The PersistentVolumeClaim must request:

- Capacity: `1Gi`
- Access mode: `ReadWriteMany`

## Prerequisites

The following tools are required:

- Docker
- Minikube
- kubectl

Verify the tools:

```bash
docker --version
minikube version
kubectl version --client
```

## Step 1: Start the Minikube Cluster

Start the existing Minikube cluster:

```bash
minikube start -p lab11 --driver=docker
```

Update the Kubernetes context:

```bash
minikube update-context -p lab11
```

Switch to the correct context:

```bash
kubectl config use-context lab11
```

Verify the current context and node:

```bash
kubectl config current-context
kubectl get nodes
```

Expected result:

```text
NAME    STATUS   ROLES           AGE   VERSION
lab11   Ready    control-plane   ...   ...
```

## Step 2: Verify the Namespace

Check whether the `ivolve` namespace exists:

```bash
kubectl get namespace ivolve
```

Create it if it does not exist:

```bash
kubectl create namespace ivolve
```

## Step 3: Create the Storage Directory

Create the application logs directory on the Minikube node:

```bash
minikube ssh -p lab11 -- \
  "sudo mkdir -p /mnt/app-logs && sudo chmod 777 /mnt/app-logs"
```

Verify the directory:

```bash
minikube ssh -p lab11 -- "ls -ld /mnt/app-logs"
```

## Step 4: Create the PersistentVolume

Create a file named `persistent-volume.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-logs-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/app-logs
    type: DirectoryOrCreate
```

The PersistentVolume uses:

```text
Capacity: 1Gi
Access mode: ReadWriteMany
Reclaim policy: Retain
Storage type: hostPath
Path: /mnt/app-logs
```

Apply the PersistentVolume:

```bash
kubectl apply -f persistent-volume.yaml
```

Expected output:

```text
persistentvolume/app-logs-pv created
```

## Step 5: Create the PersistentVolumeClaim

Create a file named `persistent-volume-claim.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-logs-pvc
  namespace: ivolve
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  volumeName: app-logs-pv
  resources:
    requests:
      storage: 1Gi
```

The PVC requests the same size and access mode as the PV:

```text
Requested storage: 1Gi
Access mode: ReadWriteMany
```

Apply the PersistentVolumeClaim:

```bash
kubectl apply -f persistent-volume-claim.yaml
```

Expected output:

```text
persistentvolumeclaim/app-logs-pvc created
```

## Step 6: Verify the PersistentVolume

Display the PersistentVolume:

```bash
kubectl get pv
```

Expected result:

```text
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
app-logs-pv   1Gi        RWX            Retain           Bound    ivolve/app-logs-pvc
```

Display detailed information:

```bash
kubectl describe pv app-logs-pv
```

Verify the following values:

```text
Capacity:       1Gi
Access Modes:   RWX
Reclaim Policy: Retain
Status:         Bound
Path:           /mnt/app-logs
```

## Step 7: Verify the PersistentVolumeClaim

Display the PVC:

```bash
kubectl get pvc -n ivolve
```

Expected result:

```text
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES
app-logs-pvc   Bound    app-logs-pv   1Gi        RWX
```

Display detailed information:

```bash
kubectl describe pvc app-logs-pvc -n ivolve
```

Verify:

```text
Status:       Bound
Volume:       app-logs-pv
Capacity:     1Gi
Access Modes: RWX
```

## Step 8: Create a Test Pod

Create a file named `test-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logs-test-pod
  namespace: ivolve
spec:
  containers:
    - name: logs-writer
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          echo "Application log created from Kubernetes Pod" > /app/logs/access.log
          sleep 3600
      volumeMounts:
        - name: app-logs
          mountPath: /app/logs
  volumes:
    - name: app-logs
      persistentVolumeClaim:
        claimName: app-logs-pvc
```

Apply the test Pod:

```bash
kubectl apply -f test-pod.yaml
```

Check its status:

```bash
kubectl get pod logs-test-pod -n ivolve
```

Wait until the Pod becomes:

```text
Running
```

## Step 9: Verify the Persistent Data

Read the log file inside the Pod:

```bash
kubectl exec -n ivolve logs-test-pod -- \
  cat /app/logs/access.log
```

Expected output:

```text
Application log created from Kubernetes Pod
```

Read the same file from the Minikube node:

```bash
minikube ssh -p lab11 -- \
  "sudo cat /mnt/app-logs/access.log"
```

Expected output:

```text
Application log created from Kubernetes Pod
```

This confirms that `/app/logs` inside the Pod is mounted to:

```text
/mnt/app-logs
```

on the Minikube node.

## Troubleshooting: ResourceQuota Error

If the test Pod is rejected with:

```text
exceeded quota: pod-quota, requested: pods=1, used: pods=2, limited: pods=2
```

The `ivolve` namespace already contains two Pods from Lab 11.

Display the existing Pods:

```bash
kubectl get pods -n ivolve
```

Delete the old test Pods:

```bash
kubectl delete pod pod-1 pod-2 -n ivolve
```

Verify that the quota usage decreased:

```bash
kubectl get resourcequota pod-quota -n ivolve
```

Then create the Lab 13 test Pod again:

```bash
kubectl apply -f test-pod.yaml
```

If Kubernetes previously rejected the Pod, this message is expected:

```text
pods "logs-test-pod" not found
```

The Pod did not exist because it was rejected before creation.

## Final Verification

Run:

```bash
kubectl get pv
kubectl get pvc -n ivolve
kubectl get pod logs-test-pod -n ivolve
kubectl describe pv app-logs-pv
kubectl describe pvc app-logs-pvc -n ivolve
```

Expected final result:

```text
PV status: Bound
PVC status: Bound
Capacity: 1Gi
Access mode: RWX
Reclaim policy: Retain
Host path: /mnt/app-logs
```

## Project Structure

```text
Lab13/
├── persistent-volume.yaml
├── persistent-volume-claim.yaml
├── test-pod.yaml
└── README.md
```

## Cleanup

Delete the test Pod:

```bash
kubectl delete -f test-pod.yaml
```

Delete the PersistentVolumeClaim:

```bash
kubectl delete -f persistent-volume-claim.yaml
```

Because the reclaim policy is `Retain`, the PersistentVolume may enter the `Released` state.

Delete the PersistentVolume:

```bash
kubectl delete -f persistent-volume.yaml
```

Delete the stored files from the node:

```bash
minikube ssh -p lab11 -- \
  "sudo rm -rf /mnt/app-logs"
```

Stop the Minikube cluster:

```bash
minikube stop -p lab11
```

## Important Note

`hostPath` is suitable for Minikube and lab environments.

In a real multi-node Kubernetes cluster, `hostPath` is tied to one node and does not provide shared storage between different nodes. Shared storage solutions such as NFS or a CSI driver should be used in production.

## Verification Checklist

- [x] Created a `1Gi` PersistentVolume.
- [x] Used `hostPath` storage.
- [x] Used `/mnt/app-logs` as the node path.
- [x] Configured `ReadWriteMany`.
- [x] Configured the `Retain` reclaim policy.
- [x] Created a PVC requesting `1Gi`.
- [x] Matched the PVC access mode with the PV.
- [x] Verified that the PV and PVC are `Bound`.
- [x] Mounted the PVC inside a test Pod.
- [x] Verified the persistent log file.
