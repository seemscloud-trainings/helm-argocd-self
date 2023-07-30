# helm-argocd-self-managed

## Initialize

```bash
helm upgrade \
    --install argo-cd argo/argo-cd \
    --namespace argocd-system \
    --version 5.41.1 \
    --values base/argo-cd/values.yaml \
    --values overlays/seemscloud/values-argo-cd.yaml
```

```bash
helm upgrade \
    --install argocd-apps argo/argocd-apps \
    --namespace argocd-system \
    --version 1.4.0 \
    --values base/argocd-apps/values.yaml \
    --values overlays/seemscloud/values-argocd-apps.yaml
```

```bash
kustomize  build overlays/seemscloud > argo.yaml
kubectl apply -f argo.yaml
```