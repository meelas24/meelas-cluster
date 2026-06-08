# Custom Grafana Dashboards

This directory contains Grafana dashboard ConfigMaps that are automatically loaded into Grafana via the **Grafana sidecar** feature (enabled by default in kube-prometheus-stack).

## Adding New Dashboards (Recommended Approach)

**Best Practice**: Create one ConfigMap per dashboard for easier management and troubleshooting.

### Steps:

1. **Download Dashboard JSON**: Get dashboard JSON from:
   - [Grafana Community Dashboards](https://grafana.com/grafana/dashboards/) - Browse thousands of pre-built dashboards
   - [Kubernetes Dashboards](https://grafana.com/grafana/dashboards/?search=kubernetes) - K8s-specific dashboards
   - [Prometheus Dashboards](https://grafana.com/grafana/dashboards/?search=prometheus) - Prometheus monitoring dashboards

2. **Create ConfigMap File**: Create a new ConfigMap YAML file in this directory:
   ```yaml
   # File: dashboards/my-dashboard-name-configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: dashboard-my-dashboard-name
     namespace: kube-prometheus-stack
     labels:
       grafana_dashboard: "1"  # 🚨 CRITICAL: This label enables auto-discovery
   data:
     my-dashboard.json: |-
       {
         "dashboard": {
           "title": "My Custom Dashboard",
           # ... paste your complete dashboard JSON here ...
         }
       }
   ```

3. **Add to Kustomization**: Add the new ConfigMap to `../kustomization.yaml`:
   ```yaml
   resources:
   - namespace.yaml
   - http-route-grafana.yaml
   - http-route-prometheus.yaml
   - dashboards/k3s-cluster-overview-configmap.yaml
   - dashboards/dashboard-16450-configmap.yaml
   - dashboards/my-dashboard-name-configmap.yaml  # 👈 Add this line
   ```

4. **Commit & Push**: ArgoCD will automatically sync and Grafana will import the dashboard

## File Organization

```
monitoring/kube-prometheus-stack/
├── kustomization.yaml                    # Main kustomization file
├── values.yaml                          # Helm values
├── namespace.yaml
├── http-route-*.yaml
└── dashboards/                          # 📁 All dashboard ConfigMaps here
    ├── README.md                        # This file
    ├── k3s-cluster-overview-configmap.yaml
    ├── dashboard-16450-configmap.yaml
    └── your-new-dashboard-configmap.yaml
```

## Naming Conventions

- **File names**: `[descriptive-name]-configmap.yaml`
- **ConfigMap names**: `dashboard-[descriptive-name]`
- **Use kebab-case**: `zfs-storage-configmap.yaml`
- **Be descriptive**: `node-exporter-full-configmap.yaml`

**Examples:**
- `k3s-cluster-overview-configmap.yaml` ✅
- `prometheus-metrics-configmap.yaml` ✅  
- `dashboard1.yaml` ❌ (not descriptive)
- `my_dashboard.yaml` ❌ (use kebab-case)

## How It Works (Technical Details)

1. **Grafana Sidecar**: The kube-prometheus-stack enables Grafana's sidecar container by default
2. **Label Watching**: Sidecar watches for ConfigMaps with label `grafana_dashboard: "1"`
3. **Auto-Discovery**: Sidecar automatically mounts dashboard JSON from matching ConfigMaps
4. **Live Reload**: Changes to ConfigMaps trigger dashboard updates in Grafana
5. **Datasource Mapping**: Dashboards automatically use the configured Prometheus datasource

## Current Dashboards

- **k3s-cluster-overview-configmap.yaml** - Basic K3s cluster metrics (CPU, memory)
- **dashboard-16450-configmap.yaml** - Community dashboard (Grafana ID 16450, revision 3)

## Troubleshooting

### Dashboard Not Appearing?
1. ✅ Check label: `grafana_dashboard: "1"` (exact match)
2. ✅ Check namespace: `kube-prometheus-stack`
3. ✅ Check ConfigMap is in kustomization.yaml resources
4. ✅ Verify ArgoCD has synced successfully
5. ✅ Check Grafana sidecar logs: `kubectl logs -n kube-prometheus-stack deployment/prometheus-grafana -c grafana-sc-dashboard`

### JSON Format Issues?
- Ensure valid JSON (no trailing commas, proper quotes)
- Use `|` or `|-` for multiline YAML strings
- Escape any `{{` template variables as `{{ "{{" }}`

## Requirements Checklist

✅ **Label**: `grafana_dashboard: "1"`  
✅ **Namespace**: `kube-prometheus-stack`  
✅ **Valid JSON**: Proper Grafana dashboard format  
✅ **Unique name**: Avoid ConfigMap name conflicts  
✅ **In kustomization**: Added to resources list  
✅ **Committed**: Changes pushed to git for ArgoCD sync
