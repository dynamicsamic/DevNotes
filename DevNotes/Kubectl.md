#### Install `kubectl` and managed by `yc`
```bash
# To install kubectl follow the instructions listed here: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

# Verify kubernetes installed successfully
>>> kubectl version --client
... Client Version: v1.31.0
... Kustomize Version: v5.4.2

# Next install the yc cli tool from: https://yandex.cloud/ru/docs/cli/quickstart#install

# Verify yc installed successfully
yc --version

# Now obtain the OAuth-token via https://yandex.cloud/ru/docs/iam/concepts/users/accounts#passport

# Then start interactive session to setup yc cli tool 
yc init
# The process is divided into 4 steps
# 1. Provide the OAuth-token
# 2. Choose the cloud that your user is regitered on
# 3. Choose the folder (kind of project)
# 4. Choose a default compute zone (you can ignore this step by providing typing n)

# Now verify that everything was setup
yc config list
>>> token: ...
cloud-id: ...
folder-id: ...

# Create a configuration file for you kubectl.
# The cluster-id may be found on the YandexCloud dashboard.
yc managed-kubernetes cluster get-credentials <cluster-id> --external

# If you have multiple clusters you can run this command separately for each cluster.
yc managed-kubernetes cluster get-credentials <cluster-id-1> --external
yc managed-kubernetes cluster get-credentials <cluster-id-2> --external

# Each cluster name will be stored as a context in ~/.kube/config file.
# You can switch contexts however you wish.
kubectl config use-context <context_name>
```
---
#### Get pod names assigned to a namespace
```bash
kubectl get pods -n <namespace>
```
---
#### Connect to a pod
```bash
# First get the pod name from the command below.
kubectl exec -itn <namespace> <pod_name> -- bash
```
---
#### Copy a file to and from a pod
```bash
# Copy to local machine
kubectl cp <namespace>/<pod-name>:/path/to/file.txt /local/file-path.txt

# Copy to pod
kubectl cp /local/file-path.txt <namespace>/<pod-name>:/path/to/file.txt 
```
---
