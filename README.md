Directory Structure

argo-grafana-loki-demo/
├── argo-apps/
│   ├── grafana/
│   │   ├── grafana-values.yaml
│   ├── loki/
│   │   ├── loki-values.yaml
│   └── grafana-agent/
│       ├── grafana-agent-values.yaml
├── manifests/
│   ├── test-log-generator-deployment.yaml
└── README.md

Step-by-Step Setup Guide

1. Set Up ArgoCD
Install ArgoCD on Minikube cluster.

# Create a namespace for ArgoCD
`kubectl create namespace argocd`

# Install ArgoCD
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

# Expose the ArgoCD API server
`kubectl port-forward svc/argocd-server -n argocd 8080:443 &` # add the & if you want it to run in background


2. Access ArgoCD UI & Login via CLI

# Get the ArgoCD admin password
`kubectl get secrets argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode ; echo`

Log in using admin as the username and the password retrieved above

# login via cli 
`argocd login localhost:8080` # this allows user to deploy from cli rather than UI

3. Create Helm Values Files
Create the necessary Helm values files for Loki, Grafana, and Grafana Agent. 

And your directory structure should look like this:

argo-grafana-loki-demo/
├── argo-apps/
│   ├── grafana/
│   │   ├── grafana-values.yaml
│   ├── loki/
│   │   ├── loki-values.yaml
│   └── grafana-agent/
│       ├── grafana-agent-values.yaml
├── manifests/
│   ├── test-log-generator-deployment.yaml
└── README.md

4. Push changes to Git

git init
git add .
git commit -m "Initial commit for Grafana and Loki setup"
git remote add origin <your-repo-url>
git push -u origin master

5. Create ArgoCD Applications

# Create the Loki application
argocd app create loki \
    --repo https://github.com/jkwarteng1/argo-grafana-loki-demo.git \
    --path argo-apps/loki \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace loki-stack \
    --sync-policy automated

# Create the Grafana application
argocd app create grafana \
    --repo https://github.com/jkwarteng1/argo-grafana-loki-demo.git \
    --path argo-apps/grafana \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace loki-stack \
    --sync-policy automated

# Create the Grafana Agent application
argocd app create grafana-agent \
    --repo https://github.com/jkwarteng1/argo-grafana-loki-demo.git \
    --path argo-apps/grafana-agent \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace loki-stack \
    --sync-policy automated

# Sync applications (As needed)
argocd app sync loki
argocd app sync grafana
argocd app sync grafana-agent

6. Verify Deployment 
# check the Argocd UI

https://localhost:8080/

7. Port-Forward to Grafana
kubectl port-forward svc/loki-grafana -n loki-stack 3000:80 & 

8. Delete and CleanUp Setup 

# Delete the namespaces
kubectl delete namespace loki-stack
kubectl delete namespace argocd

# Confirm that resources are deleted
kubectl get all --all-namespaces

# Delete ArgoCD applications without removing the resources
argocd app delete loki --cascade=true
argocd app delete grafana --cascade=true
argocd app delete grafana-agent --cascade=true
argocd app delete log-generator --cascade=true 