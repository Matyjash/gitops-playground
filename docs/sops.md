# SOPS secret management

SOPS + age encrypts secrets in Git. Argo CD's CMP sidecar decrypts them during
rendering. [`.sops.yaml`](../.sops.yaml) defines file rules and public recipients;
private keys belong outside Git, with a secure backup.

## Plugin contract

Applications select `spec.source.plugin.name: sops-kustomize-v1`.
The [plugin](../bootstrap/argocd-sops/plugin-configmap.yaml) runs Kustomize, then
appends the decrypted `database-secret.sops.yaml` from the application root.
Keep that file out of Kustomize resources and specify its final name and namespace.
Other paths and KSOPS generators require configuration changes.

## Bootstrap

Install Argo CD first. Set `SOPS_AGE_KEY_FILE` to the matching private key, check the
kubectl context, and run from the repository root. Skip Secret creation if the
correct key is already installed.

```sh
kubectl -n argocd create secret generic sops-age \
  --from-file=keys.txt="$SOPS_AGE_KEY_FILE"
kubectl apply -f bootstrap/argocd-sops/plugin-configmap.yaml
kubectl -n argocd patch deployment argocd-repo-server --type=strategic \
  --patch-file bootstrap/argocd-sops/repo-server-patch.yaml
kubectl -n argocd rollout status deployment/argocd-repo-server
```

Restore the key and plugin before syncing dependent applications after a rebuild.
Preserve the patch and align the sidecar version during Argo CD upgrades.
Restart repo-server after plugin configuration changes.

## Operations

- Edit with `sops edit <file>`; verify with `sops decrypt <file> > /dev/null`.
  Commit only encrypted secrets; application-specific fields belong in app docs.
- Rotate recipients with `sops updatekeys <file>` and install the new private key.
  Verify decryption before retiring old keys; retain them for historical revisions.
  This does not rotate application passwords.
- Decrypted secrets reach Argo CD and its cache. Restrict access to Argo CD/Redis,
  Kubernetes Secrets, and repositories using the plugin.

References: [SOPS](https://getsops.io/docs/),
[Argo CD CMP](https://argo-cd.readthedocs.io/en/stable/operator-manual/config-management-plugins/).
