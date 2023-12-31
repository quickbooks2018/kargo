# Kargo ArgoCD

- Single Kind Cluster
```bash
#!/bin/bash
# Purpose: Kubernetes Cluster
# extraPortMappings allow the local host to make requests to the Ingress controller over ports 80/443
# node-labels only allow the ingress controller to run on a specific node(s) matching the label selector
# https://kind.sigs.k8s.io/docs/user/ingress/


######################################
# Docker & Docker Compose Installation
######################################
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
rm -f get-docker.sh


#######################
# Kubectl Installation
#######################
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
kubectl version --client

####################
# Helm Installation
####################
# https://helm.sh/docs/intro/install/

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
cp /usr/local/bin/helm /usr/bin/helm
rm -f get_helm.sh
helm version

###################
# Kind Installation
###################
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.0/kind-linux-amd64

# Latest Version
# https://github.com/kubernetes-sigs/kind
# curl -Lo ./kind "https://kind.sigs.k8s.io/dl/v0.9.0/kind-$(uname)-amd64"
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.9.0/kind-linux-amd64
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.8.1/kind-linux-amd64
chmod +x ./kind

mv ./kind /usr/local/bin

#############
# Kind Config
#############
#cat << EOF > kind-config.yaml
#kind: Cluster
#apiVersion: kind.x-k8s.io/v1alpha4
#networking:
#  apiServerAddress: 0.0.0.0
#  apiServerPort: 8443
#EOF
  
# kind create --name cloudgeeks cluster --config kind-config.yaml --image kindest/node:v1.21.10
  
  # ssh -N -L 8443:0.0.0.0:8443 cloud_user@d8d0041c.mylabserver.com
  
 # export KUBECONFIG=".kube/config"
  
 ####################  
 # Multi-Node Cluster
 ####################
 cat > kind-config.yaml <<EOF
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: 0.0.0.0
  apiServerPort: 8443
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
        eviction-hard: "memory.available<5%"
        system-reserved: "memory=1Gi"
        kube-reserved: "memory=1Gi"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        eviction-hard: "memory.available<5%"
        system-reserved: "memory=1Gi"
        kube-reserved: "memory=1Gi"
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        eviction-hard: "memory.available<5%"
        system-reserved: "memory=1Gi"
        kube-reserved: "memory=1Gi"
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        eviction-hard: "memory.available<5%"
        system-reserved: "memory=1Gi"
        kube-reserved: "memory=1Gi"        
EOF
  
 kind create --name cloudgeeks cluster --config kind-config.yaml --image kindest/node:v1.28.0
 
 
  export KUBECONFIG=".kube/config"
 
#End of Kind Cluster 
```
- Mustiple Kind Cluster
```bash
#!/bin/bash
# Purpose: Kubernetes Cluster
# extraPortMappings allow the local host to make requests to the Ingress controller over ports 80/443
# node-labels only allow the ingress controller to run on a specific node(s) matching the label selector
# https://kind.sigs.k8s.io/docs/user/ingress/


# Add a new cluster in argocd kind
# docker inspect other-cluster-name
#  update kubeconfig ---> with ip address of docker container 6443, sample configmap added in README.md

######################################
# Docker & Docker Compose Installation
######################################
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
rm -f get-docker.sh


#######################
# Kubectl Installation
#######################
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
kubectl version --client

####################
# Helm Installation
####################
# https://helm.sh/docs/intro/install/

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
cp /usr/local/bin/helm /usr/bin/helm
rm -f get_helm.sh
helm version

###################
# Kind Installation
###################
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.0/kind-linux-amd64

# Latest Version
# https://github.com/kubernetes-sigs/kind
# curl -Lo ./kind "https://kind.sigs.k8s.io/dl/v0.9.0/kind-$(uname)-amd64"
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.9.0/kind-linux-amd64
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.8.1/kind-linux-amd64
chmod +x ./kind

mv ./kind /usr/local/bin

###########
# Cluster 1
###########
#############
# Kind Config
#############
cat << EOF > kind1-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
 apiServerAddress: 0.0.0.0
 apiServerPort: 8443
EOF

###########
# Cluster 2
###########
#############
# Kind Config
#############
cat << EOF > kind2-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
 apiServerAddress: 0.0.0.0
 apiServerPort: 8444
EOF
  
###########
# Cluster 3
###########
#############
# Kind Config
#############
cat << EOF > kind3-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
 apiServerAddress: 0.0.0.0
 apiServerPort: 8445
EOF

# kind create --name cloudgeeks cluster --config kind-config.yaml --image kindest/node:v1.21.10
  
  # ssh -N -L 8443:0.0.0.0:8443 cloud_user@d8d0041c.mylabserver.com
  
 # export KUBECONFIG=".kube/config"
  

  
kind create --name cloudgeeks-local-1 cluster --config kind1-config.yaml --image kindest/node:v1.28.0

kind create --name cloudgeeks-local-2 cluster --config kind2-config.yaml --image kindest/node:v1.28.0

kind create --name cloudgeeks-local-3 cluster --config kind3-config.yaml --image kindest/node:v1.28.0

 
export KUBECONFIG=".kube/config"
 
#End of Kind Cluster
```
- https://kargo.akuity.io/how-to-guides/installing-kargo

- prerequisites
```bash
You will need:

Helm: These instructions were tested with v3.11.2.
A Kubernetes cluster with cert-manager and Argo CD pre-installed. These instructions were tested with:
Kubernetes: v1.25.3
cert-manager: v1.11.5
Argo CD: v2.8.3 
```

- Helm Installation latest version
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh --version v3.11.2
./get_helm.sh
```

- Install ArgoCD
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm repo ls
helm search repo argocd
helm search repo argocd --versions
helm show values argo/argo-cd --version 5.50.1
mkdir -p argocd/values
helm show values argo/argo-cd --version 5.50.1 > argocd/values/argocd.yaml
helm upgrade --install argocd -n argocd --create-namespace argo/argo-cd --version 5.50.1 -f argocd/values/argocd.yaml --wait

helm upgrade --install argocd -n argocd --create-namespace argo/argo-cd --version 5.50.1  --wait
```

- Argocd add multiple kind cluster networking
```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJV0VCWDFaWVY1Nll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQX
hNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1EVXhOalEzTVRWYUZ3MHpNekV4TURJeE5qVXlNVFZhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJ
QkR3QXdnZ0VLCkFvSUJBUUR5R2FHLzBoMjRDRi9VTjdBVzRpQWtNaHQzV3ZDRXJnYjg1REZVb1c2SXRlWi9aUnU5cTNvZnVaalUKWm9DYlQzVEgvVjNzbFVXeVFCaHFMbGNBR1dFS2NScVhsc0ZmQWRuSE5ES2
FiZ1QwNFRsVjdzc2hYV24veXBxMQpWV1JLQjVBWGlPU3ROTzFWeE9ZdVRTMFQ0Zi92bWIzQURVQ3VFVzFHd1ZKeUdXVkZOanZEazZVU1J1WFluM0xvCndsVUhYTm5OT2xxM3pUVSs2STVTenBUN0c4OW11eWZ6
NUVXNlNsdk5jS1ZEemNldFBGRzMyaEVjS2F3a1NXYmcKck90d1dvYXo5eXduWFliUUhHMnFpNnB3aWNObTRVVjRYTWdjOWtUSTBKUkpTWWJvNXhVbUlUYk12MDVMbWtrTQp2NG0ycmY2UG1EUFRoMEZHRjNmc2
l5cVViai81QWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJSM0N1M1F0a1hlaEtLSVA5NU5saDV5Uy9QNWJqQVYKQmdOVkhSRUVE
akFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQk50N2xuQ3llUwp1QUdsWFFOVWI0NEExck1vc0RUcGJVRGtvSGJJV0tHWm12ZmVBRVhWYm5Ha1F2cWtBNUFqcHErdTVKU0dCQV
p3ClEzYTd2SlFlRFRMMlM2YWFhQUROUUxlUmVQYVdVeWVlTUtnMnU1b1VEZUdyM0hVQ1gzb3hneU80TGNlWk1LcFgKcnk3MjhDT2FzazNIVDFrcTVqYmZpbEpsZWpoRUUwNEkvYTYya3M0VURHaCtsZHkzMGFt
MnhqbDdtRHRjQm1mUwpZb3B2RlcxenpLdGhKZy8zU3Y2aDFGVnZ1Ym9SSEQwYkltZXYzZHVuYjVOc3h1djh6ZVB0aWlGdmZHS1JWaTRoCmFtSG4yQk5RbE9yMEdGQ0hnOTIwRjhTa214WHdkUU1uSm9zSmtuQ0
pKSmFka3pocHZwVmlrMUFoeTk5Q0ZaMTcKaEdVR0tSaFI2cjVsCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://0.0.0.0:8443
  name: kind-cloudgeeks-local-1
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJTWVRUUFhaTBRVkF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQX
hNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1EVXhOalEzTkRaYUZ3MHpNekV4TURJeE5qVXlORFphTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJ
QkR3QXdnZ0VLCkFvSUJBUUNsajkyQTZMNjVFbWZQSmtvNU85Q2JqOW1HRnZmcEk4VEU3dFpadlFHVWphV0JOMm8zMWxhOVcxWVMKYWMyeitZRTN0WEd4a28wTGdHYkcxWU9SMEZDbzBnMy9MVHJWNk93bU1Sb2
R1SU1TaWZhWU50bkMrd0RiN2hKKwpEdVNkVVpSRDA5T1VZQjNQdURNeWllR280YkhBYlg1bXdmYmViNE1FbXNtaElTMktaUzZ5Z3VtY2J6WklXZ1ZzCi91Y0FNUitBNTdFUWppUWI5ZW5NUHkwZzAxQ2VVWXdr
a1owdTZsNFB5WDZuTm1xKzNVVDdwVUdRdTh3UzNyaVAKSjBhVUFrN2E2VFR5K1lZRzBrKzNlc08vK0FZanh2V1g3bUhRV3d5bTRUOWZiUEhZZmxDK1V3RGx5aWZ3QzBXYQozT2pOK2tsMm02TkpPM3BhcjNnSW
FOOGtrTXhIQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUa3kzMjByRFVldGtWYXl1Yk9NR3JaZ1JMUDZUQVYKQmdOVkhSRUVE
akFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQm41RzRsOVN6Mgp3VzFOOHBaNStMZjZvSCtEOGV3ZExKckJFTTRSMXlVbXhpZzIrK21wT3hKUHVQMWFNNVJ5ZFFsQ3dDOGFzTH
BBClN0Y2REclAwMGtKN2VsRVh2bnJ5ZzQ3MHZpVDJJTVdBMk94TlRzRXZIUnRmSGlIUnVzdUxYU3JQSE96VUVZdHcKR3NTQURJZzJseEI0NUxOcGR5cnNYVmE5QytodGwyZ1lZN2JZUmM0bXF6YndTMjJReDlO
a2dQYVNUaWVSbWxIUQo0b0E3bTFzdGk1K3pCdlliNkd6aWRMSkZZU1M2SkdUNURHSTkwTXBDZmwrSDU0UHA0OHhqR1ZvZWhLV01iSTA4CnN3ZkZJOHNrWnZKSGwwTlhkN2FUZFZoZVM2eTZwYjFEck51SjBMOU
RrRkswK2d1cldIeGJWY0REREY1WWwrbGkKcFdOMXp2ZE1hbVcvCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://172.18.0.2:6443
  name: kind-cloudgeeks-local-2
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJRS9ya1RFL1V4UzR3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQX
hNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1EVXhOalE0TWpGYUZ3MHpNekV4TURJeE5qVXpNakZhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJ
QkR3QXdnZ0VLCkFvSUJBUUNvVXZ3elhYckMxVmRzYVE3SmZnckpyODZya1E5cUJuVEd6SjlmMG95V2dSV21qdThscUY3YjkvRXAKZVF2YWRTd0lQY2xWamNDUDJVZC8xcXlKbXd2alpWRFRJVTRvOGdQSEp6Sm
JHcG0xaThNNkJUR0tsQ2pMTkhYUgpjSEs2ZTY3ejRNQi96SU9HdFNHTnNTbyt5MWlPaWxLMU5ZNnRySHpNVGRLSE5MRUQyVkZmN1liZjEzSFVmN3ZtCmlFT2NvTU92RmlOdTBLbWZ1WEhFK0ZzMzB4TWlYN0xC
NTNUS1N4emFVUmFNSzNEaHg3dHdRT1VyNStaYllCRm8KMHNYS3VpZTBsT3JKbW1MUi9Va3BscmZOY2ZmWjVyM1FPd0h6TUx3TnFud2FwWnFKVXh3T3dtRGdvRmU3YXFrSwp5M2xodzVadjlLQ0ZOcnF5RjVTL0
JZZ0NlSzRiQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRdG44a1cxMnppUnBOT0pCNWZxWkQ2V09qZ2FUQVYKQmdOVkhSRUVE
akFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ0x1bXhKWWpRVQp1ank2Z1huU2VjbXlYVnVpS1VuZnBFeE1IVU1LUXg1NnhGUHpSS2RWbkpnL1hRclBpemhCM3R0M25aUDdoSX
NwCmR1ZW9mb3pHYzhmMThPWVk0emdwMEVhOTJOMkt5Q2d6aU1RUnpVa3RwMDl0RzRZdktnaEpNQXZSK1hhaEthcVAKYmdOV1k3M1ZqeDc0L2cvdS9RL251SlRlQkZ2OG4rSU9aZTVrOXRoYU8xWk05RzBDOXl5
MWFQeEZRV0MweWxkbgprRk1TZU4rck1ENURMdE83QUt0K1A2U0NnYzNEUWlOWHluUHREcWVHZCthVHlhNy9nRnRPRnU0MEF1RUJQWnRGCmtydW5Pdk9saCtiS2dPajZQeUFVdFBnZkp6VkQ3Z1ZkTExhcjVoaG
xwRXVyUDJJLzQ4bFdEdCswTVEwZW5YNlYKNng2ZnNNNklWMTk3Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://172.18.0.3:6443
  name: kind-cloudgeeks-local-3
contexts:
- context:
    cluster: kind-cloudgeeks-local-1
    user: kind-cloudgeeks-local-1
  name: kind-cloudgeeks-local-1
- context:
    cluster: kind-cloudgeeks-local-2
    user: kind-cloudgeeks-local-2
  name: kind-cloudgeeks-local-2
- context:
    cluster: kind-cloudgeeks-local-3
    user: kind-cloudgeeks-local-3
  name: kind-cloudgeeks-local-3
current-context: kind-cloudgeeks-local-1
kind: Config
preferences: {}
users:
- name: kind-cloudgeeks-local-1
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJWG9RZk5HM0o4Zk13RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS
2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1EVXhOalEzTVRWYUZ3MHlOREV4TURReE5qVXlNVGhhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGR
HVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTNNNkhXRXF4M0QxejN6WkgKY3lOc1krTTJvZU1rNjNJTzZ6S2poOWg0S21vblh5WFdQLy94a3cwQUVmU0dzZ
U5HRktTeGtXYXNzb0hrSEc4Zgp1NW5ObG9qTW1pS3EzdXZCYTZtM09mdjFCUlVVMTJhTFZNNUJBOThRRVJxK2t5amtLTUJKeWdhazlXOC9DMUVMCmdJZXZGek5oZnRENWpHYS9kdnZOSXZxTHkybkk3N0hoUWl
aRVlHNVVleUxIT0pqYzNlbXBVaXZGaFlaYlh0SkYKK0dOZlVZMEVpOGVFTEZVYitGazB1NS8vekxXTzFxVzFJOFhWSTVwMEgzTy90MEJxVEJvSzY5Yk51cHRnUlY4LworUmlwYzVLc0pheXRNQzljSW9qaG5qe
E5VVE1jZVROOGJyckZFbE5iM0J5eWZ5b2RtSWVkb0U4anZ5M2RhV2NGCktPdmJXUUlEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjB
UQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JSM0N1M1F0a1hlaEtLSVA5NU5saDV5Uy9QNQpiakFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBUDUxb3dhTGt0VkY5NllWR0J4dlQ4OVRNcXRrTHlBY213bDBrC
jVTNi9CZllkNUJ2VjhaSHlXdGQ3NHdzUTRnWU1qcnNzczUrYVRWdkFoeUljVXc3LzhuejczdFd1bkN4Rzlha2gKSlhvaktQY2pxYWlUL2tIZ0VQNkRRL1g1R0NKcHhEejRIN3AvVEJ1OFUyRnFSLzRzOHRDQ2Y
rMTdHNW50TW42cQphTGkydWFSWXZGOG1BMjZab1hSSElzd1U0aDF5TTg0ZjFIVXFMT1lmSzRLNG00d3pqOFUyM0xFYURxdVNJQ0tnCkdTdFhnZndKcjFibTRsVzZkeXhORXNoOW9hUm1ZdFNDVmVBaEJURjVsW
lR2WUdrNG1MMUpONjdWR2s5aDdvRTUKMnBmM1hwMW1QM21mc2lBZG5xTzlDaXVYTTV4enE4ZWlXQ2Jnd0twTlFUMkxMS3ZkR0E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBM002SFdFcXgzRDF6M3paSGN5TnNZK00yb2VNazYzSU82ektqaDloNEttb25YeVhXClAvL3hrd
zBBRWZTR3NlTkdGS1N4a1dhc3NvSGtIRzhmdTVuTmxvak1taUtxM3V2QmE2bTNPZnYxQlJVVTEyYUwKVk01QkE5OFFFUnEra3lqa0tNQkp5Z2FrOVc4L0MxRUxnSWV2RnpOaGZ0RDVqR2EvZHZ2Tkl2cUx5Mm5
JNzdIaApRaVpFWUc1VWV5TEhPSmpjM2VtcFVpdkZoWVpiWHRKRitHTmZVWTBFaThlRUxGVWIrRmswdTUvL3pMV08xcVcxCkk4WFZJNXAwSDNPL3QwQnFUQm9LNjliTnVwdGdSVjgvK1JpcGM1S3NKYXl0TUM5Y
0lvamhuanhOVVRNY2VUTjgKYnJyRkVsTmIzQnl5ZnlvZG1JZWRvRThqdnkzZGFXY0ZLT3ZiV1FJREFRQUJBb0lCQUMwVWJVVjFXb0tnZVR1bApGVWxJZmlTV2l0emFSdnRhdWZrQ3lhZytaYW9qS2c2ZTV2TUN
DZ3YvcDNnQytFenhIYW14Umg5dFd1ekc0QTkrCkVIaDRtaVNWb1ZBR3ErUC9OZW9KS0VOS1VXZk1PZ2tRQW5vNThjUWNrbWNya3FUd3dFUWhuZThGRkVDWGljT0IKdEZ0MVdDWGdjNU0wanY0L0srL2EwTExIZ
kNyMitMZmtnd09ETkt2dGlXV1NYY0NULytaZExiMUFXUjFCd0M2WgoyRmx4L29VdnYrdmFXblcwZmZjaU5EOS9zeVlOR0lXWlNWQVdqSDB0TmRDVjFvWUd1UWsxSEZvdGx2b1BvaGNvClhiY2hYWlRhdzBhUU4
vU2QySjNUZG14TDBieHc1WVNkbnlaRHI4anU3UW9ZZFZGY0xmS2g2bjJUSzZlUThiU04Kd2QxNStnRUNnWUVBNE43NUgrUmpGMkFaQ2IxNitFMUhuZURyaDVHbmhDSlZ2TzA2dzBYdS84WXRDcWdVaFVRaAo2M
WE4MFdoZTZzczN1Zm5oKzFLNlduWFN6Rit2c0lGNlZsNy9Mb21lNkxkMHo2empXQVRvTjJJM3JwbEc0b0xjCmpCTlBGVGNvaXoyTU1VQjF2eDI0SXQwdkVEMCtINUpUd2JleWMrdS8vMnZXTVdXWE4rU0xmdEV
DZ1lFQSsxK0cKdVdnTlFXQWI4YXlyZnFnOTM5QkcvUXZ2OTZnRGx5cWJsam55dW5yMEh3UGdMcFhmRm1OUGZDcGpGcFJhSGtnYQp3NEtHaUNkYlliQVZXb1V3dEgzd2JxWnlwaStrZFJFMGFJbVkrWkJRenlVR
Es0eitFV3lYN1o5SkQrZXNGZVhMCitwN3hPTHdFSFRqNSs5RGYzTFJldndkOCt2SWluRitwRzFnemhna0NnWUVBbzRLK2wyZ1VmWnpNMS81RmVtT0gKSGMrOHM3ay9tNTd6eFFxaTNmYnVFR0hqd29ueFVlMzI
xQThnQVF6cFo4VVk0a3hHKzk1dHp2VTVzSTRyVENiNQo4Ky9qa2MxVUZWZkF3bm11WGdSRGJuSW1sdml4U0dkclF4ZVBPNWYrRFRGVnpVSlgyLzRhTkkrc2R4eHhIYmZpCjIwdGRvVmpkSlRjZWFwQzZZbFAyO
HVFQ2dZRUFpcDFMb25QUlUycVhIamFVUHZ5eFdmajFIRmIzMWI1TmRJOW4KU25qYjg1OFhmUDdiSVhVOWxvL2U1ZDVOMDhhc3h5UUZ0aWdFM1pHdVgza0UzV3l5eGVUb3IydVRyelo0WHFIRgpINXc0d2UybDB
nUmI4aWtHZFQ5SHJCMzQ0UTlrb3BLY1g3QkQzb0EzV0pIeHI3Mm5wSUhGK1ZzOHZQakd3eW94CkMwWWc2MWtDZ1lCQ0R2TjFxVm9wT2pMMjAxQVpJai9uOXdya2lGdXlJRCtyVFJEUzdhczU4b1lNUS9CVXJqc
lkKcFBaMFNsUUxWcENybGVrVFJaM29QT0sweTRVRzJQMEppN0U2MWRORDU2N3hRRWJ6eXdROWx6RThXUUE3NkNrYgpPNmNrZkhOK2Qyek9CbVp4b3JONkdKbXB6cDNsbURMaTd3L1pkREFXcTJrbFFqSlFQTk1
Ubmc9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
- name: kind-cloudgeeks-local-2
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJQ0JXSUNweVB6WTR3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS
2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1EVXhOalEzTkRaYUZ3MHlOREV4TURReE5qVXlORGxhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGR
HVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTFEV1J1SitWbHVNaDBSTkYKQ3NNdkthanVrNW5wS3BMc01GR25DcnN5TGRWWXN1RG1qQTN0TjI2eVhlR3B3d
3VCZ1c0QjFUUkVYMzM3ZE1EbApmTDBUS0VUUS9vMkR6cDF1QVB2Nlpud1hTd3NqVmhZVjRBME1HcGkzOVpmN0JRenRCUUh0VjR4N0J2RFBwV0RkCkhWMSthMlRrbStyMzdHVXlnWmNzK0xqYXRFRzVNcVNYYkF
CeUpNeklZalB3NDhud0hLSmhPbXl0SHJkTGZmNXIKclFUY0xneDlZTll3NEM3NlV5bWdNRjI4R3VpRE5UWEJtemlUU1NOWHdmcEQ3dldWbklVZzhJL1Y5YThGN3I3RApaUjA1cVluaEtrV3lSVmdMZ21UVGE1d
2ZmY3pHVng4MVZzL2tQa0RzTWRVL0J5ZytvaG1vZll4L3d1Z1Y5WUliCnZOcW5oUUlEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjB
UQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JUa3kzMjByRFVldGtWYXl1Yk9NR3JaZ1JMUAo2VEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBaXd3ZFFnUSszQnBUR2JFMmRaOEFFbjFjaVVkdGgwb1FCcy8vC
jZLajdldnJrNFNKQ0N1NTdCdmx0RU1uNHFPczVFV0hsUm1tNml6QUNrWGg3Z1p2T2g5enhJa0RaZUNRT09pbEcKL2ZMSGRIMUlEaE9ubml2RlZoNmtjZjNkSkxqYlRnODE1MldyQ25CQm5KaDZZZ2JQWmJDQWJ
Ha0tSRDM2MTBmQgpPV0tIT3M5cmtjd2J4c0w4R01md3FtS2JOUE8vRGU3T3RmanJXUy8xTEZLY1FtL2Z0RGN5Lzdvb0lFblNxSnpjClFYZUk4ZXE1bGdlbVVzTHJmS3F6eDI5NE9ZSWZwMm44ZVVocVoyWEZMO
GYySkZ6Zm9jZThBN0J5RnlBclA1N0EKMCtGaERMYWoxU2VPVW5WdnZLalFiYUhPY3RLOEFpMlpjRE1uaS9MU2U2RkpOcjVLaEE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBMURXUnVKK1ZsdU1oMFJORkNzTXZLYWp1azVucEtwTHNNRkduQ3JzeUxkVllzdURtCmpBM3ROM
jZ5WGVHcHd3dUJnVzRCMVRSRVgzMzdkTURsZkwwVEtFVFEvbzJEenAxdUFQdjZabndYU3dzalZoWVYKNEEwTUdwaTM5WmY3QlF6dEJRSHRWNHg3QnZEUHBXRGRIVjErYTJUa20rcjM3R1V5Z1pjcytMamF0RUc
1TXFTWApiQUJ5Sk16SVlqUHc0OG53SEtKaE9teXRIcmRMZmY1cnJRVGNMZ3g5WU5ZdzRDNzZVeW1nTUYyOEd1aUROVFhCCm16aVRTU05Yd2ZwRDd2V1ZuSVVnOEkvVjlhOEY3cjdEWlIwNXFZbmhLa1d5UlZnT
GdtVFRhNXdmZmN6R1Z4ODEKVnMva1BrRHNNZFUvQnlnK29obW9mWXgvd3VnVjlZSWJ2TnFuaFFJREFRQUJBb0lCQUhVUklGU1RaUGlhVUh1QQpmK1FTRFNDVVd4aitIVTBwUkRiYnRLVU00Lys0NUhlNTdqVnh
NVjRWS1R6MVVxdVEyMTdGaXlsTTdqTW4wdUp6Cno2WGl3SURFaGU4SHJxRDY1RWFTdTI1eUluOTVvNExJS0VVenJXdTJ1RFU2SkhhQ3pQYW9PNERDdE5QOVlPL0EKNGZwaksyeWZlaVl4bDVmSHMvY2RxT2VGU
XRHUk54QlhySHhPOE0vNGo3dnZ2Qzl4S1I0Y2hGTGhpU1pRUmMxYgpOZ0dhR1U0Y3FOMUhCazFpNDJIMkZDQjIraVNINUl1YXQxVldYNWEvYVIvYTVpeWFxTHQ3SityNmZOSzFyZDRhCmlZZmsrVmJacE1jWUZ
tZE81bHk0bzhJZW9RMFlwZVIrZXBNc2dmR1NyQ0QweEdLa1RZVTUyeDBBdGl0Q1ZzUVEKV3dsOVFBRUNnWUVBNHF5UDRzR1ZLcGxzZlNIalZZRzczbS9QWi9WanRKTFRoWVlEYXZFTjEzUWVGMHZhV3lHQQpZY
jJKWnFPUzlCa052QTFyNjBYVlYySVljOGlHYmpEaUpMU1BxQkIrTmN2TWFZK3N3RzRVZ3FhOUxPalcwODZSCkdzZlpSemk2c2hkUnpRN0RZRHdVRTNkM3V5RFpBRHprbURtelBmZVZDWkhRVU12ekxTVHJuNFV
DZ1lFQTc2bnUKMU9YUktoL1BDa0tLd1BsT1VRTFlNajhJRUhkdUpWbVc2Yk9mZXBzM0EwTXhwdk91UEk1ZFlaaGg5M2RsWHMxRgp2clluL1VIMEJUZU5GYjRSN2pGeHhlMkZBSDh0WGJVbjl2ZU44RnBJaGFGW
UFHS29pYUoybGNSQzRTVTJKQ1JRCmtQMUlzQWpDdU03dDJQalo1MllmM0VOMi9qNHNzMjNhRzczdGFBRUNnWUJQc3dMb1pNNVE2eDhGSkZ3dFhXOHoKOTdaQ1JEcXBiQktwV1FSc25wTVNWVWFiUkZWalEzVkR
hSVFlTFpkbThrUXRBYjhYT3plWEFPdStFaHlLTSsvZgpuZ2tBdThQVW9IS2dEeDlialpqeWt6UGl6WDl6ZTZiemRwOUQ5b05XU1BLL0dkakNSbjE4bHIzbmN0WTV5aTlLCkFzZXVHeFl4Qmk3cnFibzMva3BHa
lFLQmdRQ0RFR3VNQWlKZzV3UlNtQkZKZHcxZFFERDdTWDBmaERDNFBFWkwKaCsrdkhUTDRsY2FSaHJIQ2JCaWxSckJNcFA3SzJYZEsxRk1LTFFkRjB4dFV5SjBGcGdTU09WS0M4d25jTlRXbAoyYnZGdFpuemV
raWw3VTQ5OXByalRIRkdyeExzc085ZVd5VmxIMlZkcTh4bTI3Qk8yNHFRNmxRb3RkZThTRmIrCjFOOEFBUUtCZ0JZcGxTMGM2MG9yOTA5dGdTaFp1a2RkNG9lNm45YmNiRDFRQTZBcWREYzB5RUZENTlTcXpye
UUKVU5iNFRBdEc0ZWc5RzVxSXRRenhMTndEbmNIazg2YlZ1b3ZpWmlHMUxJbmtFUW5INllqSE9XRWkvNGpocU9uYQpWVmR0WDNHQkVJcXlITEx1QlhubjAyM0x2OGVkMzc0SHJxK2NGeWdvZFNZYjQxU0NUOWZ
YCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
- name: kind-cloudgeeks-local-3
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJU0hmckpaRVNSYzB3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS
2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpFeE1EVXhOalE0TWpGYUZ3MHlOREV4TURReE5qVXpNak5hTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGR
HVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTVjWmRhNHZmQUR5VFZnRFUKR1FFMXpHZzNLNjBjb21OWE9EdklQNTI1VXY4dkJPWmU4UWNiUVFzaFQ2bzZDZ
WpVVkl3MXFKeVBuaXFwMTJvaQpMU3F4ZHRCZ2dscGg1dWRxa3p1R3FZU2pCdS92QXhCSVBZOUpVazA4ZHFyaHBGNnJUL0tRS09vdGh0MjJKQ2lzCjkvZStJRVYycm5PenZaTWRoN2pLaDVBZHZkWk5lYWdYRW9
weFAwdkxiS3IwYThkV0hrdmtyWGNML3Rmb2N1dWcKKy84WTVTa1I3aTMrZDlvMC84RkJrV2ZsZm1FbU1DUUh0N3g1QWFKQ3Y1WktqZW5MNWlNalFURWw5VERKenU3NApCcW1VMTJ5Szc3Mm81Uy8rS2FWL3VtU
mxaT0ZwTXg3RW8wTFJqSUlXY1djbkJIT29pVUx2ZkxBbUVXMzYzNUtZCmNEYmdpUUlEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjB
UQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JRdG44a1cxMnppUnBOT0pCNWZxWkQ2V09qZwphVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBRkx1UXRVSlVmaDBQWFU1MW8vcUNNZDBMcXFQVThmS0IxVlY3C
npqM3VEdlRyaTl1NFdNSjAzM0F0ZkhRejZ6QzRpMTZlVTRRVmVublZ4VUV6UHdYZHlPalIyZGZ2TGN5NmFLNUcKWmVOUzVpRi8rK3ZoZWtzMDY5ZDlMNkVWM0VGUFMrWXlGdmdPM29QYitvaVQ1WFo0VUJURHp
VVEJQYVBiR0lLVAptbXdocWMwbVlrR01rRU10UjNLZzJKK3BpbUI5MXBkcDJmRWhrKzEra1JQRGEzODh4aE1rOU1sUnkwYlFjWWVvCjZvUGxKbmlnb05YMno1ekNrVVpYamlOaTA4d25nR2hTZWc0M2h2U0FWZ
TJQYXE1SmpGSy8rUytsMmtZN0Y2bnAKZnFaMXdnRFNmeHlmbkRQNlppNVpuTkRuZVZXWXN0dUFKeXo1Tm5PMzBkYmp1NE1tMUE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBNWNaZGE0dmZBRHlUVmdEVUdRRTF6R2czSzYwY29tTlhPRHZJUDUyNVV2OHZCT1plCjhRY2JRU
XNoVDZvNkNlalVWSXcxcUp5UG5pcXAxMm9pTFNxeGR0QmdnbHBoNXVkcWt6dUdxWVNqQnUvdkF4QkkKUFk5SlVrMDhkcXJocEY2clQvS1FLT290aHQyMkpDaXM5L2UrSUVWMnJuT3p2Wk1kaDdqS2g1QWR2ZFp
OZWFnWApFb3B4UDB2TGJLcjBhOGRXSGt2a3JYY0wvdGZvY3V1ZysvOFk1U2tSN2kzK2Q5bzAvOEZCa1dmbGZtRW1NQ1FICnQ3eDVBYUpDdjVaS2plbkw1aU1qUVRFbDlUREp6dTc0QnFtVTEyeUs3NzJvNVMvK
0thVi91bVJsWk9GcE14N0UKbzBMUmpJSVdjV2NuQkhPb2lVTHZmTEFtRVczNjM1S1ljRGJnaVFJREFRQUJBb0lCQUVxcE5RdlFEQ3I0Zyt0cgpJMy9vZThHcWoxcTZ5bFlkcjJhUFRsY1ZlZlYxQTZNMDg5NmZ
yNFJvQ0cvcFlTaDlKaGsrNGVTaTBxdlRNeExTCjRyNElaRmUyQjlYelptSndDWnBEdWxMMHpVQmcwQ29QclZtTGFJaThuZ3YxSkpLVFRGa1MrVExDUFA3WXBlbUQKNmdnODBPT05qcTNLM0xtWnh2dWwyUEZKc
TlCVjUzS1lmRDVvczFqSXFleTF1MEUvVkxCd2J6WjgxOTh4TWxkQQpJUEpYYzRzbjVMbXF3cUdTZWUyS1pYWDU2SG5CcmVJWHlJL0tqbThsK0VidXpzZFdxVisrRlRKNENqUE1CZFEvCndsSVF4bThRanh6Rkt
zVXBsRUpVL0x3cmRETTRpL3U4LzI3T1Q2QnJHL0xubjM1NE12c2FXaDlhUkZUVVdjSjkKdzNwTHJMMENnWUVBNjYwR3ZDT2ZHS2R4RkJVakpOWEh4Vko3T2dYd1BIQjh2d0FBNGNrSmtJNzluek5LZUthOQpQQ
mh2dXpqcElBcDBPb0c1cSt4SWlWVXRjaFY0UllIL3NBV0h4OHdjOXdVRmRWNHJCcEJMSXJ5S3kyUGF1amM2CjdjTGp5VUtFZDljdmpOTUtWMTgrOFFONGozWFQxZTB2TUIvRjM1Z1hNM3o4UlZoY1I2OHRnck1
DZ1lFQStaY1EKRm81QkdWMmlOTVZUOHpyd2pTZ0lCS3hQZnJ3RGFPcm1JTkloNGJLb25sWnhSQTRBVkxBMUlUY3F1UXp5VXBuWgp1eGhYZEUvQXZRalBka2E0QjhXN3JBc2tRcHVDcTlpbUpaY1VIRE5ZaXliL
2NPcm5FTnY1OHZKeDEzbEY2cHIvCmRZYmg0d2JyS0FNTHg4cURZdEZRcWVHV1hlSXQveER6Q3YzeXZkTUNnWUJrMit5bzhabXh4RjdwV0U1dEFpeDMKYnVBYnFkVHhHTGJhU3RjVnlhS1ZRZ3lhYUNuenptL24
yeEZlRDNIOWExdm9lZ0lLNkpqMlpRU0k4VlZDY2VzSgoycVFNUm5jUzVBdzgrV3J5YVNndW1uL1BvVkZISmtISzVQLzA3c21nTy9BMmxVNjBxNmRWSDhZbHB5aGNRaUJTCnVSWkdwV0V3Z01tNXA2SCtiNkxMY
ndLQmdBTGw3eWpqNC91Z2E3YkRKOU5tTnM3Y3pTTUl3UytPalZlVmlyQ00KNEJuWDBqOXNiNHBEdzFzNFpKV0xKM0xZcEtPeTU2VlZoZ1p5dXFFM1RmbG9udEJ3U2xxWUVvYTNlWS8zUnc2ZQpyM3dZV0luZEh
SQTVtZzlIRHFMMGo3L1p4NmNPdjdLa3ExRFFqc0I3TUpMVVZpdzZrLzQxQVdMN3NsOEkybG1oClUwVkRBb0dBWTIxc250elZNWnF3UUpNV0prZE1OWkc2TUYwc0toRjJ2QUU0bnBaVGZQdXI0MXNTdDNQT2txa
2MKdlZZZmE2b2FvV1dXeWt2VUhGNm1wRkRJa3dVUVRZSytGZW9YSW84U05vMUxGN05tZytzU3JMaUJIUkgyWDlUYQpqOG8vQStZbktTVjBiK3AzeXNqS1hpSEE5NjYrWm8zUE5STlJQWVNtQ05MWUhsN1ozZW8
9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```

- Kubernetes Context
```bash
kubectl config get-contexts

kubectl config get-clusters

kubectl config use-context kind-cloudgeeks

kubectl config current-context

kubectl cluster-info
```

- ArgoCD Installation https://github.com/argoproj/argo-cd/releases
```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
argocd version
```

- ArgoCD Specific version Installation
```bash
export VERSION=<TAG>
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
argocd version
```

- Argocd Add kind Clusters
```bash
kubectl -n argocd port-forward svc/argocd-server --address 0.0.0.0 8091:443
```
```bash
argocd login localhost:8091 --insecure --username admin --password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
```
```bash
kubectl config get-contexts
argocd cluster add kind-cloudgeeks-local-2 --name kind-cloudgeeks-local-2 --kubeconfig=/root/.kube/config
argocd cluster add kind-cloudgeeks-local-3 --name kind-cloudgeeks-local-3 --kubeconfig=/root/.kube/config
```

- Cert Manager Installation
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm repo ls
helm search repo jetstack
helm search repo jetstack/cert-manager --versions
helm show values jetstack/cert-manager --version v1.13.2
helm show values jetstack/cert-manager --version v1.13.2 > cert-manager-values.yaml
helm upgrade --install cert-manager jetstack/cert-manager --version v1.13.1 --namespace cert-manager --set installCRDs=true --create-namespace --values cert-manager-values.yaml
helm upgrade --install cert-manager jetstack/cert-manager --version v1.13.1 --namespace cert-manager --set installCRDs=true --create-namespace
```

- KarGo Installation
```bash
helm upgrade --install kargo \
  oci://ghcr.io/akuity/kargo-charts/kargo \
  --namespace kargo \
  --create-namespace \
  --set api.adminAccount.password=admin \
  --set api.adminAccount.tokenSigningKey=iwishtowashmyirishwristwatch \
  --set api.adminAccount.enabled=true \
  --wait
```

- kargo default password
```bash
admin
```

- Kargo CLI Installation
```bash
arch=$(uname -m)
[ "$arch" = "x86_64" ] && arch=amd64
curl -L -o kargo https://github.com/akuity/kargo/releases/latest/download/kargo-$(uname -s | tr '[:upper:]' '[:lower:]')-${arch}
chmod +x kargo
mv kargo /usr/local/bin/
kargo --help
```

- Karo port-forward
```bash
kubectl -n kargo port-forward svc/kargo-api --address 0.0.0.0 8090:443
```

- https://kargo.akuity.io/quickstart/
- Karo login
```bash
kargo login https://localhost:8090 --insecure-skip-tls-verify --admin --password admin
```

- The kubectl command to merge a new kubeconfig file into an existing kubeconfig file is as follows:
```bash
KUBECONFIG=~/.kube/config:new-kube-config.yaml kubectl config view --flatten > merged-kube-config.yaml
```
- To remove a specific cluster configuration from an existing kubeconfig file, you would use kubectl config commands to unset the contexts, clusters, and users associated with that configuration. Below are the commands that you would use to remove the configuration for the cluster named kind-cloudgeeks-local-2:

```bash
kubectl config delete-cluster kind-cloudgeeks-local-2
kubectl config delete-context kind-cloudgeeks-local-2
kubectl config delete-user kind-cloudgeeks-local-2
```

- JQ utility
```bash
kubectl get secrets/argocd-initial-admin-secret -o json -n argocd | jq .data.password -r
kubectl get secrets/argocd-initial-admin-secret -o json -n argocd | jq .data.password -r | base64 --decode
```
