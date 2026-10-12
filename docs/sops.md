# SOPS secret management

Secrets are encrypted with SOPS and age before being committed to Git.
Argo CD uses KSOPS to decrypt them during Kustomize rendering.

- Before syncing applications, ensure Argo CD has KSOPS enabled and access to
  the matching private key. Restore both after rebuilding the cluster.
- Use `sops edit <file>` to edit an encrypted secret and
  `sops decrypt <file> > /dev/null` to check decryption.
- Commit only encrypted secrets. Reference them through a KSOPS generator,
  not directly as Kustomize resources.
