# Ansible Playbooks for Cloudigan Infrastructure

**Repository:** https://github.com/heybearc/ansible-playbooks  
**Semaphore UI:** https://ansible.cloudigan.net  
**Last Updated:** March 29, 2026

---

## 🚀 Quick Start

### Deploy a New Container
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=omada-controller" \
  -e "container_function=network" \
  -e "container_ip=10.92.3.16" \
  -e "container_domain=omada.cloudigan.net" \
  -e "container_port=8043"
```

### Run via Semaphore
1. Go to https://ansible.cloudigan.net
2. Navigate to **Task Templates**
3. Find "Deploy Proxmox Container"
4. Click **Run** and enter variables

---

## 📋 Available Playbooks

### Infrastructure Deployment
- **`deploy-proxmox-container.yml`** - Deploy LXC container with full automation
- **`deploy-proxmox-vm.yml`** - Deploy VM with full automation
- **`deploy-proxmox.yml`** - Unified deployment (container or VM)

### System Management
- **`system-update.yml`** - Update all packages on hosts
- **`health-check.yml`** - Monitor system health
- **`fix-python-modules.yml`** - Bootstrap Python dependencies
- **`distribute-ssh-key.yml`** - Distribute SSH keys to hosts

### Application Management
- **`nodejs-app-restart.yml`** - Restart PM2-managed Node.js apps

### Database Management
- **`postgresql-status.yml`** - Check PostgreSQL cluster health
- **`postgresql-failover.yml`** - Promote replica to primary
- **`postgresql-rejoin-old-primary.yml`** - Rejoin old primary as replica

### Automation
- **`sync-inventory.yml`** - Auto-update inventory from Netbox
- **`sync-semaphore-templates.yml`** - Auto-create Semaphore templates

---

## 🔄 Keeping Semaphore in Sync

**Problem:** Playbooks exist but aren't visible in Semaphore UI

**Solution:** Run the sync playbook

```bash
# Via Semaphore UI
1. Go to https://ansible.cloudigan.net
2. Find "Sync Semaphore Templates" task
3. Click Run

# Or via command line
cd /Users/cory/Projects/ansible-playbooks
ansible-playbook playbooks/sync-semaphore-templates.yml \
  -e "semaphore_password=YOUR_PASSWORD"
```

This will:
- Scan GitHub repo for all playbooks
- Auto-create Semaphore templates for each one
- Make them visible in the UI

**Run this whenever you add new playbooks!**

---

## 📚 Documentation

- **`README-DEPLOYMENT.md`** - Detailed deployment guide with examples
- **`TEMPLATE-SYNC.md`** - How the auto-sync system works

---

## 🎯 Common Use Cases

### Deploy Omada Controller
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=omada-controller" \
  -e "container_function=network" \
  -e "container_ip=10.92.3.16" \
  -e "container_memory=2048" \
  -e "container_cores=2" \
  -e "container_domain=omada.cloudigan.net" \
  -e "container_port=8043" \
  -e "container_ssl=true"
```

### Deploy Development Environment
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=dev-sandbox" \
  -e "container_function=dev" \
  -e "container_ip=10.92.3.110" \
  -e "container_memory=8192" \
  -e "skip_npm=true"
```

### Check System Health
```bash
ansible-playbook playbooks/health-check.yml
```

### Update All Systems
```bash
ansible-playbook playbooks/system-update.yml
```

---

## 🔗 Integration

This repo integrates with:
- **Proxmox** - Container/VM creation
- **Netbox** - IPAM registration
- **Nginx Proxy Manager** - Reverse proxy setup
- **AdGuard Home** - DNS registration
- **Prometheus** - Monitoring setup
- **Semaphore** - Web UI for playbook execution

---

## 💡 Tips

1. **Always use Semaphore UI** - Easier than CLI, has history
2. **Run sync-semaphore-templates** after adding new playbooks
3. **Check README-DEPLOYMENT.md** for detailed examples
4. **Use `--check` mode** to preview changes (CLI only)

---

## 🆘 Troubleshooting

**Playbook not in Semaphore?**
- Run `sync-semaphore-templates.yml`
- Check playbook has metadata in `semaphore-auto-template.py`

**Container deployment fails?**
- Verify IP not in use
- Check Netbox/NPM credentials
- Review Proxmox logs

**Can't find this repo?**
- Bookmark: `/Users/cory/Projects/ansible-playbooks`
- Or: `cd ~/Projects/ansible-playbooks`

---

**Repository:** https://github.com/heybearc/ansible-playbooks  
**Maintained By:** Cory Allen
