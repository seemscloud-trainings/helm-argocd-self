```bash
for i in `seq 0 2` ; do
  kubectl taint nodes "k3d-${CLUSTER_NAME}-server-$i" dedicated=control-plane:NoSchedule
done

for i in `seq 0 5` ; do
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/generic=true
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/worker=true
done

export PROJ_PATH="seemscloud-trainings/helm-argocd-self"
export BRANCH_NAME="main"

export REPO_URL_RAW="https://raw.githubusercontent.com/${PROJ_PATH}"
export REPO_URL="https://github.com/${PROJ_PATH}.git"

kubectl create namespace argocd-system

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