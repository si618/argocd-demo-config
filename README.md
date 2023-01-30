# argocd-demo-config
[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) configuration for [argocd-demo-app](https://github.com/si618/argocd-demo-app) based on [Nana Janashia's tutorial](https://youtu.be/MeU5_k9ssrs) 🙇‍♂️
## Setup

Instructions to install Kubernetes via MicroK8s, ArgoCD and a demo application running on WSL 2

### Install Ubuntu

```bash
wsl --install --distribution Ubuntu
# Verify WSL 2 is running
wsl --list
Windows Subsystem for Linux Distributions:
Ubuntu-22.04 (Default)
```

### Install SystemD

```bash
wsl
sudo nano /etc/wsl.conf

# Copy the next two lines into wsl.conf and save
[boot]
systemd=true
```
```bash
# Restart WSL
exit
wsl --shutdown
wsl
```
```bash
# Verify SystemD is running
systemctl list-unit-files --type=service
```
### Install MicroK8s
```bash
sudo snap install microk8s --classic
# Verify MicroK8s is running
microk8s status --wait-ready
# Verify Kubernetes is running
microk8s kubectl get all --all-namespaces
# Install dashboard and dns
microk8s enable dashboard dns
microk8s dashboard-proxy
# exit once
```
### Install ArgoCD

```bash
# Set MicroK8s as the current context
kubectl config use-context microk8s
# Verify MicroK8s is current context
kubectl config current-context
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
# Wait until all ArgoCD pods are running
kubectl get pods -n argocd -w
# Get the admin user password for ArgoCD UI
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
# Setup access to the ArgoCD UI
kubectl port-forward svc/argocd-server 8080:443 -n argocd
# Browse to https://127.0.0.1:8080
```

### Install argocd-demo-config

```bash
git clone https://github.com/si618/argocd-demo-config.git
cd argocd-demo-config
kubectl apply -f application.yaml
```