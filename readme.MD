# 🌐 Kubernetes Ingress with AWS ALB Controller (EKS)

## 📌 Overview

This project demonstrates how to expose Kubernetes applications using **AWS Application Load Balancer (ALB)** via the **AWS Load Balancer Controller**.

It enables **secure, scalable, and production-ready traffic routing** to multiple services using Kubernetes Ingress.

---

## 🏗️ Architecture

* Kubernetes Cluster (EKS)
* AWS ALB Controller
* ALB (Application Load Balancer)
* Applications (app1, app2)
* DNS → Routes traffic to ALB

---

## 📂 Project Structure

```id="ing123"
.
├── app1/
│   ├── Dockerfile
│   ├── manifest.yaml
│
├── app2/
│   ├── Dockerfile
│   ├── manifest.yaml
│
└── README.md
```

---

## ⚙️ Setup Instructions

### 🔹 Step 1: Enable OIDC Provider

```bash id="cmd1"
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster roboshop-dev \
  --approve
```

---

### 🔹 Step 2: Create IAM Policy

```bash id="cmd2"
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.16.0/docs/install/iam_policy.json

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam-policy.json
```

---

### 🔹 Step 3: Create IAM Role & Service Account

```bash id="cmd3"
eksctl create iamserviceaccount \
--cluster=roboshop-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

---

### 🔹 Step 4: Install ALB Controller (Helm)

```bash id="cmd4"
helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=roboshop-dev \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller
```

---

### 🔹 Step 5: Verify Installation

```bash id="cmd5"
kubectl get pods -n kube-system
```

---

## 🚀 Deployment

Apply Kubernetes manifests:

```bash id="cmd6"
kubectl apply -f manifest.yaml
```

---

## 🌐 Ingress Configuration

### Key Features:

* **Ingress Class:** ALB
* **Protocol:** HTTPS (443)
* **SSL:** AWS ACM certificate
* **Routing:** Host-based routing

```yaml id="yaml1"
alb.ingress.kubernetes.io/scheme: internet-facing
alb.ingress.kubernetes.io/target-type: ip
alb.ingress.kubernetes.io/group.name: roboshop
```

---

## 🔁 Traffic Flow

1. User hits domain (e.g., `app1.domain.com`)
2. DNS resolves to ALB
3. ALB routes request to Kubernetes service
4. Service forwards request to pods

---

## 🔐 Security

* HTTPS enabled using ACM certificate
* IAM role-based access for controller
* No direct node exposure

---

## 🚀 Benefits

* Scalable traffic routing
* Single ALB for multiple services
* Secure HTTPS communication
* Cloud-native integration with AWS

---

## 🔮 Future Improvements

* Add path-based routing
* Integrate WAF for security
* Add monitoring (CloudWatch, Prometheus)
* Automate via Terraform

---

## 👨‍💻 Author

Pavan – DevOps Engineer


Only Related to AWS LOAD BAlancer Installation
1. Install OIDC provider

```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster roboshop-dev \
    --approve
```

2. Download IAM Policy
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.16.0/docs/install/iam_policy.json
```

3. Create IAM policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

4. Create Service account and IAM role. map the above policy
```
eksctl create iamserviceaccount \
--cluster=roboshop-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<ACcount-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

### Install drivers

Install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

Add EKS chart repo
```
helm repo add eks https://aws.github.io/eks-charts
```

Install Load balancer controller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=roboshop-dev --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```

check drivers are running
```
kubectl get pods -n kube-system
```
