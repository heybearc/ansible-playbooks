# Container Decommissioning Playbook

**Purpose:** Complete removal of containers and all associated infrastructure entries  
**Playbook:** `decommission-container.yml`  
**Status:** Ready for use

---

## Overview

This playbook performs the **opposite** of the container provisioning pipeline. It completely removes a container and cleans up all infrastructure entries:

1. ✅ **Monitoring** - Removes from Prometheus targets and Loki/Promtail
2. ✅ **Backups** - Removes Proxmox backup schedule
3. ✅ **Reverse Proxy** - Removes NPM proxy host entry
4. ✅ **IPAM** - Removes VM, interfaces, and IPs from Netbox
5. ✅ **Container** - Stops and destroys Proxmox LXC container
6. ✅ **Documentation** - Updates APP-MAP.md, infrastructure-spec.md, and creates BookStack checklist
7. ⚠️ **DNS** - Manual removal required from DC-01 (Windows DNS Manager)

---

## Quick Start

### Basic Usage

```bash
cd /Users/cory/Projects/ansible-playbooks

ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115"
```

### Preview Changes (Check Mode)

```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  --check
```

### Force Decommission (Skip Confirmation)

```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  -e "force_decommission=true"
```

---

## Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `container_name` | Container hostname | `test-app` |
| `container_ctid` | Proxmox CTID | `115` |

---

## Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `force_decommission` | `false` | Skip confirmation prompt |
| `proxmox_host` | `10.92.0.10` | Proxmox host IP |
| `netbox_url` | `http://10.92.3.18` | Netbox API URL |
| `npm_url` | `http://10.92.3.3:81` | NPM API URL |

---

## Environment Variables Required

Set these in your environment or `.env` file:

```bash
# Netbox
export NETBOX_TOKEN="your_netbox_api_token"

# NPM (Nginx Proxy Manager)
export NPM_USER="admin@cloudigan.com"
export NPM_PASSWORD="your_npm_password"

# Or load from .env file
source ~/Projects/homelab-nexus/.env
```

---

## Prerequisites

### 1. Ansible Inventory

Ensure your inventory has these hosts defined:

```ini
[proxmox]
prox ansible_host=10.92.0.10

[monitoring]
prometheus ansible_host=10.92.3.50
loki ansible_host=10.92.3.51
```

### 2. SSH Access

Ensure you have SSH access to:
- Proxmox host (`prox`)
- Prometheus server
- Loki/Promtail server

### 3. API Credentials

Ensure you have valid credentials for:
- Netbox API token
- NPM admin credentials
- AdGuard Home admin credentials

---

## Usage Examples

### Example 1: Decommission Development Container

```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115"
```

**Output:**
```
TASK [Display decommission plan]
ok: [localhost] => {
    "msg": [
        "==========================================",
        "CONTAINER DECOMMISSION PLAN",
        "==========================================",
        "Container Name: test-app",
        "Container CTID: 115",
        ...
    ]
}

TASK [Confirm decommission (interactive)]
[Confirm decommission (interactive)]
Type 'yes' to proceed with decommissioning test-app (CT115):
yes

TASK [Remove from Prometheus targets]
changed: [localhost]

TASK [Remove from Proxmox backup schedule]
changed: [localhost]

...

TASK [Destroy Proxmox container]
changed: [localhost]

PLAY RECAP
localhost : ok=15 changed=6 unreachable=0 failed=0
```

### Example 2: Preview Decommission (Dry Run)

```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  --check
```

This shows what **would** be done without actually executing changes.

### Example 3: Batch Decommission Multiple Containers

```bash
# Create a script for multiple containers
cat > decommission-batch.sh << 'EOF'
#!/bin/bash

CONTAINERS=(
  "test-app-1:115"
  "test-app-2:116"
  "old-service:120"
)

for container in "${CONTAINERS[@]}"; do
  name="${container%%:*}"
  ctid="${container##*:}"
  
  echo "Decommissioning $name (CT$ctid)..."
  ansible-playbook playbooks/decommission-container.yml \
    -e "container_name=$name" \
    -e "container_ctid=$ctid" \
    -e "force_decommission=true"
  
  echo "---"
done
EOF

chmod +x decommission-batch.sh
./decommission-batch.sh
```

---

## Decommission Reports

After each run, two detailed reports are generated:

### 1. Decommission Report
**Location:** `/tmp/decommission-<container_name>-<ctid>.md`

**Example Report:**

```markdown
# Container Decommission Report

**Date:** 2026-03-23T18:52:00Z
**Container:** test-app
**CTID:** 115
**IP Address:** 10.92.3.50

## Actions Completed

### Infrastructure Cleanup
- [x] Removed from Prometheus monitoring
- [x] Removed backup schedule
- [x] Removed NPM reverse proxy
- [x] Removed from Netbox IPAM
- [x] Destroyed Proxmox container
- [ ] **MANUAL:** Remove DNS record from DC-01 for test-app.cloudigan.net

### Documentation Updates
- [x] Updated APP-MAP.md in control plane
- [x] Updated infrastructure-spec.md
- [x] Created BookStack update checklist

## Status

✅ **Container successfully decommissioned**

## Manual Follow-up Required

1. **Review BookStack Documentation:**
   - See: `/tmp/bookstack-update-test-app-115.md`
   - Update container inventory, runbooks, and client docs

2. **Commit Documentation Changes:**
   ```bash
   cd ~/Projects/homelab-nexus
   git add .cloudy-work/_cloudy-ops/context/APP-MAP.md
   git add documentation/infrastructure-spec.md
   git commit -m "docs: remove test-app (CT115) after decommission"
   git push
   ```

3. **Remove DNS Record from DC-01:**
   - Connect to DC-01 Windows Server
   - Open DNS Manager
   - Remove A record: test-app.cloudigan.net → 10.92.3.50

4. **Verify Removal:**
   - Check Grafana dashboards (no metrics from 10.92.3.50)
   - Verify DNS resolution removed: `nslookup test-app.cloudigan.net`
   - Check NPM proxy list: https://npm.cloudigan.net
   - Verify Netbox IPAM: https://netbox.cloudigan.net

---

**Executed by:** cory
**Playbook:** decommission-container.yml
```

### 2. BookStack Update Checklist
**Location:** `/tmp/bookstack-update-<container_name>-<ctid>.md`

This file contains a checklist of manual documentation updates needed in BookStack.

---

## Safety Features

### 1. Interactive Confirmation

By default, the playbook requires manual confirmation:

```
Type 'yes' to proceed with decommissioning test-app (CT115):
```

### 2. Check Mode Support

Use `--check` to preview changes without executing:

```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  --check
```

### 3. Non-Fatal Failures

If a component fails (e.g., NPM not accessible), the playbook continues with other steps and reports the issue.

### 4. Detailed Logging

All actions are logged with clear status messages:
- ✅ Success
- ⚠️ Warning (non-fatal)
- ❌ Error (fatal)

---

## Troubleshooting

### Issue: "Required variables missing"

**Cause:** Missing `container_name` or `container_ctid`

**Solution:**
```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115"
```

### Issue: "Could not remove from Netbox"

**Cause:** Invalid or missing `NETBOX_TOKEN`

**Solution:**
```bash
export NETBOX_TOKEN="your_valid_token"
```

### Issue: "Could not remove from NPM"

**Cause:** Invalid NPM credentials or NPM not accessible

**Solution:**
```bash
export NPM_USER="admin@cloudigan.net"
export NPM_PASSWORD="your_password"

# Test NPM access
curl -u "$NPM_USER:$NPM_PASSWORD" http://10.92.3.33:81/api/tokens
```

### Issue: "Container not found in Proxmox"

**Cause:** Container already destroyed or wrong CTID

**Solution:** Verify CTID:
```bash
ssh prox "pct list"
```

### Issue: "SSH connection failed to prox"

**Cause:** SSH access not configured or wrong inventory

**Solution:** Verify SSH access:
```bash
ssh prox "hostname"
```

---

## Integration with Provisioning Pipeline

This playbook is the **reverse** of the provisioning pipeline:

| Provisioning Step | Decommissioning Step |
|-------------------|----------------------|
| 1. Auto-assign CTID | 7. Destroy container |
| 2. Create LXC container | 7. Stop & destroy |
| 3. Register in Netbox | 6. Remove from Netbox |
| 4. Create NPM proxy | 5. Remove NPM proxy |
| 5. Add DNS record | 4. Remove DNS record |
| 6. Install monitoring | 2. Remove monitoring |
| 7. Configure backups | 3. Remove backups |

---

## Best Practices

### 1. Always Preview First

```bash
# Preview changes
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115" \
  --check

# Then execute
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115"
```

### 2. Backup Before Decommission

```bash
# Create manual backup before decommissioning
ssh prox "vzdump {{ container_ctid }} --mode snapshot --compress zstd"
```

### 3. Verify Container is Not Critical

```bash
# Check what's running
ssh prox "pct exec {{ container_ctid }} -- systemctl list-units --type=service --state=running"

# Check dependencies
grep -r "{{ container_name }}" /Users/cory/Projects/homelab-nexus/documentation/
```

### 4. Review Decommission Report

```bash
# After decommission, review the report
cat /tmp/decommission-{{ container_name }}-{{ container_ctid }}.md
```

### 5. Update Documentation

After decommissioning, update:
- `infrastructure-spec.md`
- `APP-MAP.md` (control plane)
- Remove service runbooks
- Update network diagrams

---

## Comparison: Provisioning vs Decommissioning

### Provisioning Pipeline
```bash
./scripts/provisioning/provision-container.sh \
  --name test-app \
  --function dev \
  --ip 10.92.3.50
```

**Creates:**
- ✅ Proxmox container
- ✅ Netbox IPAM entries
- ✅ NPM proxy host
- ✅ DNS record
- ✅ Monitoring targets
- ✅ Backup schedule

### Decommissioning Playbook
```bash
ansible-playbook playbooks/decommission-container.yml \
  -e "container_name=test-app" \
  -e "container_ctid=115"
```

**Removes:**
- ✅ Monitoring targets
- ✅ Backup schedule
- ✅ DNS record
- ✅ NPM proxy host
- ✅ Netbox IPAM entries
- ✅ Proxmox container

---

## Future Enhancements

- [ ] Add support for container export before destruction
- [ ] Integration with backup verification
- [ ] Slack/Teams notification on decommission
- [ ] Audit log integration
- [ ] Support for VM decommissioning (not just LXC)
- [ ] Automatic documentation updates
- [ ] Integration with change management system
- [ ] Rollback capability (restore from backup)

---

## Related Documentation

- **Provisioning Pipeline:** `/Users/cory/Projects/homelab-nexus/scripts/provisioning/README.md`
- **Container Naming Standard:** `documentation/container-naming-standard.md`
- **Infrastructure Spec:** `documentation/infrastructure-spec.md`
- **Netbox IPAM:** `documentation/NETBOX-IPAM-SETUP.md`

---

**Last Updated:** 2026-03-23  
**Maintained By:** Infrastructure Team  
**Status:** Ready for production use
