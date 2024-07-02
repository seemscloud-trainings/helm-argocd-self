```bash
export CLUSTER_NAME=seems
```

```bash
k3d cluster delete "${CLUSTER_NAME}"
```

```bash
export K3D_FIX_MOUNTS=1

k3d cluster create \
  --servers 3 \
  --agents 6 \
  --k3s-arg "--cluster-cidr=10.100.0.0/16@server:*" \
  --k3s-arg "--service-cidr=10.200.0.0/16@server:*" \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --no-lb ${CLUSTER_NAME}
```

```bash
for i in `seq 0 2` ; do
  kubectl taint nodes "k3d-${CLUSTER_NAME}-server-$i" dedicated=control-plane:NoSchedule
done

for i in `seq 0 5` ; do
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/generic=true
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/worker=true
done
```

```bash
export PROJ_PATH="seemscloud-trainings/helm-argocd-self"
export BRANCH_NAME="main"

export REPO_URL_RAW="https://raw.githubusercontent.com/${PROJ_PATH}"
export REPO_URL="https://github.com/${PROJ_PATH}.git"

kubectl create namespace argocd-system

helm repo add argo https://argoproj.github.io/argo-helm

helm upgrade \
    --install argocd argo/argo-cd \
    --namespace argocd-system \
    --version 5.51.4 \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/base/stacks/argocd/argocd/values.yaml"

helm upgrade \
    --install argocd-apps argo/argocd-apps \
    --namespace argocd-system \
    --version 1.4.0 \
    --values "${REPO_URL_RAW}/${BRANCH_NAME}/base/stacks/argocd/argocd-apps/values.yaml"

kubectl apply -f <(kustomize build "${REPO_URL}/overlays/seemscloud?ref=${BRANCH_NAME}")
```
