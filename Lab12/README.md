# Lab 12: Managing Configuration and Sensitive Data with ConfigMaps and Secrets

## Objective

This lab demonstrates how to manage application configuration and sensitive database credentials in Kubernetes using:

- ConfigMap for non-sensitive configuration.
- Secret for sensitive credentials.
- Base64 encoding for Secret values.

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

Update the Kubernetes context if the API server port changes:

```bash
minikube update-context -p lab11
```

Switch to the correct context:

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

## Step 2: Verify the Namespace

Check whether the `ivolve` namespace exists:

```bash
kubectl get namespace ivolve
```

If it does not exist, create it:

```bash
kubectl create namespace ivolve
```

## Step 3: Encode the Secret Values

The Secret values must be stored using Base64 encoding.

Encode the database user password:

```bash
printf %s 'ivolve_password' | base64
```

Output:

```text
aXZvbHZlX3Bhc3N3b3Jk
```

Encode the MySQL root password:

```bash
printf %s 'root_password' | base64
```

Output:

```text
cm9vdF9wYXNzd29yZA==
```

Using `printf` prevents adding a newline character to the encoded value.

## Step 4: Create the ConfigMap

Create a file named `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  namespace: ivolve
data:
  DB_HOST: mysql-service
  DB_USER: ivolve_user
```

The ConfigMap stores non-sensitive values:

- `DB_HOST`: MySQL Kubernetes Service hostname.
- `DB_USER`: Database user used by the application.

Apply the ConfigMap:

```bash
kubectl apply -f configmap.yaml
```

Expected output:

```text
configmap/mysql-config created
```

## Step 5: Create the Secret

Create a file named `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: ivolve
type: Opaque
data:
  DB_PASSWORD: aXZvbHZlX3Bhc3N3b3Jk
  MYSQL_ROOT_PASSWORD: cm9vdF9wYXNzd29yZA==
```

The Secret stores:

- `DB_PASSWORD`: Password for `ivolve_user`.
- `MYSQL_ROOT_PASSWORD`: MySQL root password.

Apply the Secret:

```bash
kubectl apply -f secret.yaml
```

Expected output:

```text
secret/mysql-secret created
```

## Step 6: Verify the ConfigMap

Display the ConfigMap:

```bash
kubectl get configmap mysql-config -n ivolve
```

Display its details:

```bash
kubectl describe configmap mysql-config -n ivolve
```

Expected values:

```text
DB_HOST: mysql-service
DB_USER: ivolve_user
```

Display the ConfigMap as YAML:

```bash
kubectl get configmap mysql-config -n ivolve -o yaml
```

## Step 7: Verify the Secret

Display the Secret:

```bash
kubectl get secret mysql-secret -n ivolve
```

Expected output:

```text
NAME           TYPE     DATA   AGE
mysql-secret   Opaque   2      ...
```

Display Secret details:

```bash
kubectl describe secret mysql-secret -n ivolve
```

The command displays the Secret keys and sizes without printing the decoded passwords.

## Step 8: Decode the Secret Values

Decode `DB_PASSWORD`:

```bash
kubectl get secret mysql-secret -n ivolve \
  -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
echo
```

Expected output:

```text
ivolve_password
```

Decode `MYSQL_ROOT_PASSWORD`:

```bash
kubectl get secret mysql-secret -n ivolve \
  -o jsonpath='{.data.MYSQL_ROOT_PASSWORD}' | base64 --decode
echo
```

Expected output:

```text
root_password
```

## Step 9: Final Verification

Run:

```bash
kubectl get configmap,secret -n ivolve
```

Then run:

```bash
kubectl describe configmap mysql-config -n ivolve
kubectl describe secret mysql-secret -n ivolve
```

Expected final result:

```text
ConfigMap: mysql-config
Secret: mysql-secret
Namespace: ivolve
ConfigMap keys: 2
Secret keys: 2
```

## Project Structure

```text
Lab12/
├── configmap.yaml
├── secret.yaml
└── README.md
```

## Cleanup

Delete the ConfigMap:

```bash
kubectl delete configmap mysql-config -n ivolve
```

Delete the Secret:

```bash
kubectl delete secret mysql-secret -n ivolve
```

Stop the Minikube cluster:

```bash
minikube stop -p lab11
```

## Security Note

Base64 is encoding, not encryption. Base64 values can be decoded easily.

Real production passwords should not be committed to a public Git repository.

## Verification Checklist

- [x] Verified the `ivolve` namespace.
- [x] Created the `mysql-config` ConfigMap.
- [x] Stored `DB_HOST` in the ConfigMap.
- [x] Stored `DB_USER` in the ConfigMap.
- [x] Created the `mysql-secret` Secret.
- [x] Encoded Secret values using Base64.
- [x] Stored `DB_PASSWORD` in the Secret.
- [x] Stored `MYSQL_ROOT_PASSWORD` in the Secret.
- [x] Verified the ConfigMap.
- [x] Verified and decoded the Secret values.
