# gramola

# Install ArgoCD using the operator

# Add plugin section to ArgoCD Custom Resource

```sh
kubectl patch argocd argocd-cluster -n openshift-gitops --patch "$(cat ./argocd/plugins/argocd-kustomized-helm-plugin.yaml)" --type=merge
```

# Adjust permissions of Service Account

```sh
kubectl apply -f util/argocd-service-account-permissions.yaml
```

# Log in ArgoCD with CLI

```sh
argocd-login.sh
```

# Register repos

```sh
argocd-register-repos.sh

argocd repo list
```

# Add ArgoCD Project definitions

```sh
kubectl apply -f argocd/projects/project-dev.yml
kubectl apply -f argocd/projects/project-test.yml

argocd proj list
```

# Create Root Apps

NOTE: https://argoproj.github.io/argo-cd/user-guide/helm/

```sh
export BASE_REPO_URL=https://gitlab.cm.jccm.es/web-movilidad/portales-cloud.git
#helm template ./argocd/root-apps/ --name-template portales-cloud-root-apps --set baseRepoUrl=${BASE_REPO_URL} | kubectl apply -f -

cat <<EOF | kubectl apply -n openshift-gitops -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gramola-root-app
  namespace: openshift-gitops
  labels:
    argocd-root-app: "true"
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    namespace: openshift-gitops
    name: in-cluster
  project: default
  syncPolicy:
    automated: {}
  source:
    helm:
      parameters:
        - name: baseRepoUrl
          value: ${BASE_REPO_URL}
    path: argocd/root-apps
    repoURL: ${BASE_REPO_URL}
    targetRevision: HEAD
EOF

```

# Sync Root Apps alone

```sh
argocd app sync gramola-root-app-dev
argocd app sync gramola-root-app-test
argocd app sync gramola-root-app-test-roks
```

# Sync apps manually

```sh
argocd app sync economiacircular-app-dev
argocd app sync economiacircular-app-test
argocd app sync economiacircular-app-test-roks
```

# Sync children apps (app of apps)

```sh
argocd app sync -l app.kubernetes.io/instance=gramola-root-app
argocd app sync -l app.kubernetes.io/instance=gramola-root-app-dev
argocd app sync -l app.kubernetes.io/instance=gramola-root-app-test
argocd app sync -l app.kubernetes.io/instance=gramola-root-app-test-roks
```

# AUX
git clone https://oauth2:1AbCDeF_g2HIJKLMNOPqr@gitlab.com/yourusername/project.git project