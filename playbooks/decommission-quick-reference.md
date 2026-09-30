# Container Decommissioning - Quick Reference

## Basic Command

```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=CONTAINER_NAME" \
  -e "container_ctid=CTID"
```

## Common Examples

### Preview Changes (Dry Run)
```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  --check
```

### Decommission with Confirmation
```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115"
```

### Force Decommission (No Prompt)
```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  -e "force_decommission=true"
```

## What Gets Removed

1. ✅ Prometheus monitoring targets
2. ✅ Loki/Promtail log collection
3. ✅ Proxmox backup schedules
4. ✅ Nginx Proxy Manager entries
5. ✅ Netbox IPAM (VM, interfaces, IPs)
6. ✅ Proxmox LXC container
7. ✅ APP-MAP.md entries (control plane)
8. ✅ infrastructure-spec.md entries
9. ⚠️ DNS (manual removal from DC-01 required)

## Required Environment Variables

```bash
export NETBOX_TOKEN="your_token"
export NPM_USER="admin@cloudigan.com"
export NPM_PASSWORD="your_password"

# Or load from .env
source ~/Projects/homelab-nexus/.env
```

## Required Inventory Hosts

Your inventory must have:
- `prox` - Proxmox host
- `prometheus` - Prometheus server
- `loki` - Loki/Promtail server

## Output

Reports saved to:
- `/tmp/decommission-{container_name}-{ctid}.md` - Full decommission report
- `/tmp/bookstack-update-{container_name}-{ctid}.md` - BookStack update checklist

## Post-Decommission Tasks

1. Review BookStack update checklist
2. Commit documentation changes:
   ```bash
   cd ~/Projects/homelab-nexus
   git add .cloudy-work/_cloudy-ops/context/APP-MAP.md
   git add documentation/infrastructure-spec.md
   git commit -m "docs: remove {container_name} (CT{ctid}) after decommission"
   git push
   ```
3. Remove DNS record from DC-01 (Windows DNS Manager)
4. Verify removal in Grafana/DNS/NPM/Netbox

## Safety Features

- ✅ Interactive confirmation required (unless forced)
- ✅ Check mode support (`--check`)
- ✅ Non-fatal failures (continues on errors)
- ✅ Detailed decommission report

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Missing variables | Add `-e "container_name=X" -e "container_ctid=Y"` |
| Netbox errors | Check `NETBOX_TOKEN` environment variable |
| NPM errors | Check `NPM_USER` and `NPM_PASSWORD` |
| SSH errors | Verify inventory has `prox` host defined |

## Full Documentation

See: `playbooks/DECOMMISSION-README.md`
