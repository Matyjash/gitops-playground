# Local bootstrap

Requires Docker, k3d, kubectl, and the private age key matching the encrypted
secrets. Run from the repository root.

1. Create the cluster with HTTP and HTTPS exposed locally:

   ```sh
   k3d cluster create playground --image rancher/k3s:v1.35.5-k3s1 \
     -p '127.0.0.1:80:80@loadbalancer' -p '127.0.0.1:443:443@loadbalancer' --wait
   kubectl config use-context k3d-playground
   ```

2. Install [Argo CD](https://github.com/argoproj/argo-cd/releases/tag/v3.5.5):

   ```sh
   kubectl create namespace argocd
   kubectl apply -n argocd --server-side --force-conflicts \
     -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.5.5/manifests/install.yaml
   ```

3. Enable KSOPS and provide the existing private key (replace the path):

   ```sh
   kubectl -n argocd create secret generic sops-age \
     --from-file=keys.txt=/absolute/path/to/keys.txt
   kubectl -n argocd patch configmap argocd-cm --type=merge \
     --patch-file bootstrap/argocd/argocd-cm.yaml
   kubectl -n argocd patch deployment argocd-repo-server --type=strategic \
     --patch-file bootstrap/argocd/repo-server-ksops-patch.yaml
   kubectl -n argocd rollout status deployment --timeout=300s
   ```

4. Start GitOps reconciliation and check application status:

   ```sh
   kubectl apply -f bootstrap/root-application.yaml
   kubectl -n argocd get applications
   ```

Once synced, open https://argocd.localhost or http://pollandgo.localhost.
Log in to Argo CD as `admin` using the initial password:

```sh
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 --decode
```

See [secret management](sops.md) and [local TLS setup](ingress.md).
