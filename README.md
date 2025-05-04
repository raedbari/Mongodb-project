#  Using Kubernetes to Deploy a MongoDB Database with a Mongo Express Web-Based Admin Interface in AWS

## 🛠️ Requirements

1. Launch 2 EC2 instances:
   - One as the **master**
   - The other as **node1**
---

## ⚙️ Kubeadm Not Installed?

If you haven’t installed Kubernetes yet using `kubeadm` on the master and worker nodes, follow the installation steps here:

🔗 [Go to the install-k8s repository](https://github.com/raedbari/install-k8s)

---

2. SSH into the master node using **MobaXterm**

3. Ensure the Kubernetes cluster (using `kubeadm`) is installed and configured on both the master and node1

---
## 🔐 Step 1: Create an IAM User with EBS & EFS Permissions

## 🔑 Step 2: Generate Access Key for the User

## 🎯 Step 3: Attach Policies

Attach the following AWS managed policies to your IAM user:
- `AmazonEBSCSIDriverPolicy`
---

## 🛡️ Step 4: Create IAM Role with Required Inline Policy
`AmazonEBSCSIDriverPolicy`
---

## 📦 Step 5: Install AWS CSI Drivers

### EBS CSI Driver

```bash
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.26"
```
---

## 🔐 Step 6: Create Kubernetes Secret for Access Keys

```bash
kubectl create secret generic raed-project \
  --namespace kube-system \
  --from-literal "key_id=YOUR_ACCESS_KEY_ID" \
  --from-literal "access_key=YOUR_SECRET_ACCESS_KEY"
```
Make sure to update deployments to reference this secret.

### Patch EBS CSI Controller Deployment

```bash
kubectl edit deployment ebs-csi-controller -n kube-system
```

Under `containers.env`, add:

```yaml
- name: AWS_ACCESS_KEY_ID
  valueFrom:
    secretKeyRef:
      key: key_id
      name: raed-project
      optional: true
- name: AWS_SECRET_ACCESS_KEY
  valueFrom:
    secretKeyRef:
      key: access_key
      name: raed-project
      optional: true
```

Base64 encode a password (for example):

```bash
echo -n 'rraaeedd' | openssl base64
# Output: cnJhYWVlZGQ=
```
---

## 📁 Project Directory

```bash
mkdir Mongodb-project && cd Mongodb-project
```
---
###  Mongodb Deployment Files

- [mongodb-secret.yaml](./Yaml-Files/mongodb-secret.yaml)
- [mongodb-sc.yaml](./Yaml-Files/mongodb-sc.yaml)
- [mongodb-pvc.yaml](./Yaml-Files/mongodb-pvc.yaml)
- [mongodb-app.yaml](./Yaml-Files/mongodb-app.yaml)
- [mongodb-svc.yaml](./Yaml-Files/mongodb-svc.yaml)
---
### Mongoexpress YAML Files:

- [mongo-express-app.yaml](./Yaml-Files/mongo-express-app.yaml)
- [mongo-express-svc.yaml](./Yaml-Files/mongo-express-svc.yaml)

ubuntu@master:~/mongodb-project$ kubectl get svc
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes          ClusterIP      10.96.0.1        <none>        443/TCP          18d
mongo-express-svc   NodePort   10.108.153.177   <pending>     8081:31000/TCP       13s
mongodb-svc         ClusterIP      10.99.140.21     <none>        27017/TCP        4m30s

## try to access the App in your browser 
# http://<Master-Node-Public-IP>:31000
