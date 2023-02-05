# argocd-demo-config
[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) configuration for [argocd-demo-app](https://github.com/si618/argocd-demo-app) based on [Nana Janashia's tutorial](https://youtu.be/MeU5_k9ssrs) 🙇‍♂️
## Setup
Instructions to install Kubernetes via MicroK8s on three nodes (one control plane/worker, two worker-only), ArgoCD and a demo application running on Ubuntu via [Multipass](https://multipass.run)

### Install Multipass
Windows
```bash
winget install multipass
```
Linux
```bash
sudo snap install multipass
```

### Create Virtual Machines (VM)

```bash
# Create VMs for a Kubernetes control plane and two worker nodes
multipass launch 22.10 --name kube-control-plane-1 --memory 2G --disk 8G --cpus 2
multipass launch 22.10 --name kube-worker-node-1 --memory 2G --disk 8G --cpus 2
multipass launch 22.10 --name kube-worker-node-2 --memory 2G --disk 8G --cpus 2
```

### Verify VMs are running

```bash
multipass list
```

### Install MicroK8s on VMs
```bash
multipass shell kube-control-plane-1
```
```bash
sudo snap install microk8s --classic

# Permit traffic between the VM and host
sudo iptables -P FORWARD ACCEPT

# Add user to microk8s group
sudo usermod -a -G microk8s $USER

# Grant access to .kube for caching
sudo chown -f -R $USER ~/.kube

# Add: alias kubectl='microk8s kubectl'
nano ~/.bash_aliases
```

... repeat for `kube-worker-node-1` and `kube-worker-node-2`

### Install the Kubernetes dashboard and make it accessible to the VM host
```bash
# Verify MicroK8s is running
microk8s status --wait-ready

# Install dashboard and allow access to vm from the host
microk8s enable dashboard dns ingress host-access storage

# Verify Kubernetes and its dashboard are running
kubectl get all --all-namespaces

# Change dashboard from ClusterIP to LoadBalancer
kubectl patch svc kubernetes-dashboard -n kube-system -p '{"spec": {"type": "LoadBalancer"}}'

# Copy kube config to vm host to load in dashboard
microk8s config
# Other options: https://microk8s.io/docs/addon-dashboard
```

### Join worker nodes to control-plane
```bash
multipass shell kube-control-plane-1

microk8s add-node
```

Copy output from `add-node` for worker
```bash
multipass shell kube-worker-node-1

microk8s join <IP Address>:25000/.../... --worker
```

... repeat for `kube-worker-node-2`

### Load the Kubernetes dashboard and verify nodes are joined
```bash
multipass shell kube-control-plane-1

# Get the kubernetes-dashboard port number
kubectl -n kube-system get services
```
... browse to Kubernetes dashboard at `https://<IP address>:<kubernetes-dashboard-port-number>`

### Install ArgoCD

```bash
multipass shell kube-control-plane-1

microk8s enable community
microk8s enable argocd

# Wait until all ArgoCD pods are running
kubectl get pods -n argocd -w

# Get the initial admin user password for ArgoCD
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo

# Change argocd-dashboard from ClusterIP to LoadBalancer
kubectl patch svc argo-cd-argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get the argocd-dashboard port number
kubectl -n argocd get services
```

### Install demo application

```bash
git clone https://github.com/si618/argocd-demo-config.git
cd argocd-demo-config

# Setup ingress to allow access to ArgoCD UI from VM host
# https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts
kubectl apply -f ingress.yaml

# Deploy demo app to cluster
kubectl apply -f application.yaml
```
... browse to ArgoCD UI at `https://<IP Address>:<argocd-dashboard-port-number>`
