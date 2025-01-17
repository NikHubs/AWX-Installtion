
# Installation Guide for AWX with K3s

> **_Note:_** Need this guide in German? [Click here](README-de.md)

## Preparations

### 1. Update your Linux system
To ensure your system is ready, bring it up to date. Run the following command:

```bash
apt-get update && apt-get upgrade -y
```

---

## Installing K3s

### Requirements
- **Internet connection:** An active internet connection is required for installation.
- **Root user:** Ideally, perform the installation as the root user.

### Install K3s
Download and install K3s using the following command:

```bash
curl -sfL https://get.k3s.io | sh -
```

### Check status
After installation, you can check the status of K3s:

```bash
systemctl status k3s
```

Verify that the nodes are correctly set up:

```bash
kubectl get nodes
```

### Success message
With these steps, you have successfully installed K3s, and it should now be running.

---

## Installing the AWX Operator

### Requirements
The **AWX Operator** is required for installation starting from AWX version **<18.0.0**.

### Installation steps
1. Clone the AWX Operator repository:
   ```bash
   git clone https://github.com/ansible/awx-operator.git
   ```

2. Create a namespace for AWX:
   ```bash
   export NAMESPACE=awx
   kubectl create ns ${NAMESPACE}
   ```

3. Navigate to the AWX Operator directory:
   ```bash
   cd awx-operator/
   ```

4. Find the latest release tag and check it out:
   ```bash
   RELEASE_TAG=`curl -s https://api.github.com/repos/ansible/awx-operator/releases/latest | grep tag_name | cut -d '"' -f 4`
   echo $RELEASE_TAG
   git checkout $RELEASE_TAG
   ```

5. Deploy the AWX Operator:
   ```bash
   make deploy
   ```

### Check status
The installation may take a few minutes. Check the status of the pods using:

```bash
kubectl get pods -n awx
```

---

## Installing AWX

### 1. Create a Persistent Volume Claim
Create the file `public-static-pvc.yaml`:

```bash
vim public-static-pvc.yaml
```

Insert the following content:
```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: public-static-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 20Gi
```

Save the file with `ESC` and `:wq`.

### 2. Create the AWX deployment
Create the file `awx-instance-deployment.yml`:

```bash
vim awx-instance-deployment.yml
```

Insert the following content:
```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  projects_persistence: true
  projects_storage_access_mode: ReadWriteOnce
  web_extra_volume_mounts: |
    - name: static-data
      mountPath: /var/lib/projects
  extra_volumes: |
    - name: static-data
      persistentVolumeClaim:
        claimName: public-static-data-pvc
```

Save the file with `ESC` and `:wq`.

### 3. Apply the configuration
Apply the configurations and check the status:

```bash
kubectl apply -f public-static-pvc.yaml -n awx
kubectl apply -f awx-instance-deployment.yml -n awx
kubectl get pods -n awx
```

### 4. Monitor status live
Track the progress live with the following command (exit with `CTRL+C`):

```bash
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx -w
```

---

## Accessing AWX

### 1. Check service details
Find the IPs and ports using the following command:

```bash
kubectl get svc -n awx
```

Example output:
```plaintext
[root@awx-server]# kubectl get svc -n awx
NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
awx-operator-controller-manager-metrics-service   ClusterIP   10.43.219.250   <none>        8443/TCP                195d
awx-postgres-13                                   ClusterIP   None            <none>        5432/TCP                195d
awx-service                                       NodePort    10.43.154.175   <none>        80:30080/TCP            195d 
```

- **Port 30080** is used for the web interface (Port 80 is reserved for K3s).

### 2. Change port (optional)
If you want to change the port, use the following command:

```bash
kubectl patch svc awx-service -n awx --type='json' -p='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080}]'
```

### 3. Retrieve the admin password
Find the default password with the following command:

```bash
kubectl -n awx get secret awx-admin-password -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
```

### 4. Login
- Open the AWX web interface at: `http://<SERVER-IP>:<PORT>`.
- The default username is `admin`.
- The password was retrieved in the previous step.

---

## Summary
With this guide, you have:
- **Installed and configured K3s.**
- **Deployed the AWX Operator.**
- **Successfully installed AWX and accessed its interface.**

If you need further support, check out the [K3s documentation](https://k3s.io) or the [AWX Operator GitHub page](https://github.com/ansible/awx-operator).

### Sources
Parts of this guide are based on the tutorial by [RPI4Cluster](https://rpi4cluster.com/awx-install/).
