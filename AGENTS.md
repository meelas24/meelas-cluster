# Agent Instructions

## Guardrails

- **Never run destructive commands** on the cluster (delete PVCs, PVs, namespaces, CRDs, storage classes, or `helm uninstall`) without asking for approval first.
- **No manual `kubectl` changes to live state** unless the resource cannot be managed via GitOps (e.g., patching a PV's `reclaimPolicy`). Even then, prefer a GitOps-compatible alternative and explain why.
- **Always commit and push** configuration changes. ArgoCD auto-syncs — there is rarely a reason to `kubectl apply` or `kubectl delete` anything.
- **No `kubectl delete pod`** or `kubectl rollout restart` unless the user explicitly approves it (config changes from GitOps will naturally roll on the next sync, but some DaemonSets/pods need a manual kick — ask).
- **Do not modify ArgoCD Application CRDs directly** (e.g., `kubectl patch application`) unless the user asks you to. Use Git commits instead.
- **When in doubt, ask.** The cluster has live data (immich photos, databases, etc.). Destroying a PVC or PV will lose data.

## Cluster Context

- K3s + Cilium (CNI + Gateway API + kube-proxy replacement)
- ArgoCD with two ApplicationSets (`infrastructure/*`, `applications/*`)
- Storage: zfs-localpv (OpenEBS CSI), `storageClassName: zfs-localpv`
- Secrets: SOPS (age encryption), ExternalSecrets with ClusterSecretStore
- DNS: external-dns + Cloudflare (CNAME to `external.saleem.us` via tunnel)
- Ingress: Cilium Gateway API (currently broken — L7 proxy has 0 active redirects; workaround is cloudflared direct routing)
- Node IP changed from 10.0.50.x to 192.168.68.x — do not hardcode `--node-ip`
- Two overlapping ArgoCD releases: `argo-cd-*` (bootstrap Helm) and `argocd-*` (managed via AppSet)

## Known Issues

- **Cilium gateway L7 proxy is broken** — do not assume HTTPRoute-based ingress works. Cloudflared routes directly to service ClusterIPs as a workaround.
- **StorageClass `zfs-localpv` has `reclaimPolicy: Delete`** — you can patch individual PVs to `Retain` if needed. The field is immutable on StorageClass, so it cannot be changed via GitOps without deleting/recreating it (destructive, requires approval).
