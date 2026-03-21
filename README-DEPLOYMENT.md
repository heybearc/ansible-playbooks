# Proxmox Deployment Playbooks

Comprehensive Ansible playbooks for deploying LXC containers and VMs to Proxmox with full automation integration.

## Features

- ✅ **Auto-assign CTID/VMID** based on function category
- ✅ **Netbox IPAM integration** - Automatic device and IP registration
- ✅ **Nginx Proxy Manager** - Automatic reverse proxy creation
- ✅ **DNS registration** - AdGuard Home integration
- ✅ **Monitoring setup** - node_exporter installation
- ✅ **Backup configuration** - Proxmox backup scheduling
- ✅ **Idempotent** - Safe to re-run
- ✅ **Dry-run support** - Preview changes before applying

## Quick Start

### Deploy a Container

```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=bookstack" \
  -e "container_function=core" \
  -e "container_ip=10.92.3.50" \
  -e "container_domain=bookstack.cloudigan.net" \
  -e "container_port=80" \
  -e "container_memory=4096"
```

### Deploy a VM

```bash
ansible-playbook playbooks/deploy-proxmox-vm.yml \
  -e "vm_name=windows-dc" \
  -e "vm_function=infrastructure" \
  -e "vm_ip=10.92.4.2" \
  -e "vm_iso=local:iso/windows-server-2022.iso" \
  -e "vm_memory=8192" \
  -e "vm_cores=4" \
  -e "vm_disk=100"
```

### Using the Unified Playbook

```bash
# Deploy container
ansible-playbook playbooks/deploy-proxmox.yml \
  -e "deployment_type=container" \
  -e "container_name=plane" \
  -e "container_function=core" \
  -e "container_ip=10.92.3.51"

# Deploy VM
ansible-playbook playbooks/deploy-proxmox.yml \
  -e "deployment_type=vm" \
  -e "vm_name=ubuntu-server" \
  -e "vm_function=application" \
  -e "vm_ip=10.92.3.100" \
  -e "vm_template=9000"
```

## Container Deployment

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `container_name` | Container hostname | `bookstack` |
| `container_function` | Function category (see below) | `core` |
| `container_ip` | IP address | `10.92.3.50` |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `container_ctid` | Auto-assigned | Specific CTID |
| `container_memory` | 2048 | RAM in MB |
| `container_cores` | 2 | CPU cores |
| `container_disk` | 32 | Disk size in GB |
| `container_privileged` | false | Privileged container |
| `container_template` | Ubuntu 22.04 | LXC template |
| `container_domain` | - | Domain for NPM proxy |
| `container_port` | 80 | Backend port |
| `container_ssl` | false | Enable SSL |
| `skip_netbox` | false | Skip Netbox registration |
| `skip_npm` | false | Skip NPM proxy |
| `skip_dns` | false | Skip DNS registration |
| `skip_monitoring` | false | Skip monitoring setup |
| `skip_backup` | false | Skip backup config |

### Container Function Categories

| Function | CTID Range | Description |
|----------|------------|-------------|
| `bot` | 100-109 | Discord/Telegram bots |
| `dev` | 110-119 | Development environments |
| `media` | 120-129 | Media servers (Plex, Jellyfin) |
| `core` | 130-139 | Core infrastructure services |
| `network` | 140-149 | Network services (DNS, proxy) |
| `monitoring` | 150-159 | Monitoring stack |
| `storage` | 160-169 | Storage services |
| `security` | 170-179 | Security services |
| `utility` | 180-189 | Utility services |

## VM Deployment

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `vm_name` | VM hostname | `windows-dc` |
| `vm_function` | Function category (see below) | `infrastructure` |
| `vm_ip` | IP address | `10.92.4.2` |
| `vm_iso` OR `vm_template` | ISO path or template VMID | `local:iso/ubuntu.iso` or `9000` |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `vm_id` | Auto-assigned | Specific VMID |
| `vm_memory` | 4096 | RAM in MB |
| `vm_cores` | 2 | CPU cores |
| `vm_disk` | 50 | Disk size in GB |
| `vm_bridge` | vmbr0 | Network bridge |
| `vm_cloudinit` | false | Use cloud-init |
| `vm_autostart` | true | Start VM after creation |
| `vm_domain` | - | Domain for NPM proxy |
| `vm_port` | 80 | Backend port |
| `vm_ssl` | false | Enable SSL |
| `skip_netbox` | false | Skip Netbox registration |
| `skip_npm` | false | Skip NPM proxy |
| `skip_dns` | false | Skip DNS registration |
| `skip_backup` | false | Skip backup config |

### VM Function Categories

| Function | VMID Range | Description |
|----------|------------|-------------|
| `infrastructure` | 100-119 | Core infrastructure (DC, DNS) |
| `application` | 120-139 | Application servers |
| `database` | 140-159 | Database servers |
| `development` | 160-179 | Development VMs |
| `testing` | 180-199 | Testing/QA VMs |

## Examples

### MSP Platform Deployment

#### BookStack (Documentation Hub)
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=bookstack" \
  -e "container_function=core" \
  -e "container_ip=10.92.3.50" \
  -e "container_domain=docs.cloudigan.net" \
  -e "container_port=80" \
  -e "container_memory=4096" \
  -e "container_cores=2" \
  -e "container_disk=50"
```

#### Plane (Project Management)
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=plane" \
  -e "container_function=core" \
  -e "container_ip=10.92.3.51" \
  -e "container_domain=projects.cloudigan.net" \
  -e "container_port=3000" \
  -e "container_memory=4096" \
  -e "container_cores=2" \
  -e "container_disk=50"
```

#### Authentik (Identity Provider)
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=authentik" \
  -e "container_function=security" \
  -e "container_ip=10.92.3.70" \
  -e "container_domain=auth.cloudigan.net" \
  -e "container_port=9000" \
  -e "container_ssl=true" \
  -e "container_memory=4096" \
  -e "container_cores=2" \
  -e "container_disk=50"
```

### Development Environment

```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=dev-sandbox" \
  -e "container_function=dev" \
  -e "container_ip=10.92.3.110" \
  -e "container_memory=8192" \
  -e "container_cores=4" \
  -e "container_disk=100" \
  -e "skip_npm=true" \
  -e "skip_dns=true"
```

### Windows Server VM

```bash
ansible-playbook playbooks/deploy-proxmox-vm.yml \
  -e "vm_name=windows-app-01" \
  -e "vm_function=application" \
  -e "vm_ip=10.92.4.10" \
  -e "vm_iso=local:iso/windows-server-2022.iso" \
  -e "vm_memory=16384" \
  -e "vm_cores=8" \
  -e "vm_disk=200"
```

### Clone from Template

```bash
ansible-playbook playbooks/deploy-proxmox-vm.yml \
  -e "vm_name=ubuntu-web-01" \
  -e "vm_function=application" \
  -e "vm_ip=10.92.3.120" \
  -e "vm_template=9000" \
  -e "vm_cloudinit=true" \
  -e "vm_domain=web01.cloudigan.net"
```

## Environment Variables

Set these in your environment or `.env` file:

```bash
export NETBOX_TOKEN="your-netbox-api-token"
export NPM_PASSWORD="your-npm-password"
export ADGUARD_PASSWORD="your-adguard-password"
```

## Integration Details

### Netbox
- Automatically registers virtual machines
- Creates IP address records
- Links to Proxmox cluster
- Adds custom fields (CTID/VMID)

### Nginx Proxy Manager
- Creates reverse proxy hosts
- Configures SSL (if enabled)
- Sets up WebSocket support
- Enables HTTP/2 and HSTS

### AdGuard Home
- Adds DNS A records
- Format: `hostname.cloudigan.net → IP`

### Prometheus Monitoring
- Installs node_exporter
- Exposes metrics on port 9100
- Auto-discovered by Prometheus

### Proxmox Backup
- Daily snapshots
- Zstandard compression
- Stored on local storage
- Retention: 7 days (configurable)

## Troubleshooting

### CTID/VMID Already in Use
The playbook auto-assigns the next available ID in the function range. If all IDs are taken, it will fail with an error message.

### Netbox Registration Fails
- Verify `NETBOX_TOKEN` is set
- Check Netbox is reachable at `http://10.92.3.18`
- Ensure cluster ID exists in Netbox

### NPM Proxy Creation Fails
- Verify `NPM_PASSWORD` is set
- Check NPM is reachable at `http://10.92.3.3:81`
- Ensure domain is not already registered

### Container Won't Start
- Check Proxmox logs: `ssh prox "pct status <CTID>"`
- Verify IP is not in use
- Check template exists

## Advanced Usage

### Skip All Integrations
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=test" \
  -e "container_function=dev" \
  -e "container_ip=10.92.3.200" \
  -e "skip_netbox=true" \
  -e "skip_npm=true" \
  -e "skip_dns=true" \
  -e "skip_monitoring=true" \
  -e "skip_backup=true"
```

### Custom Template
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=debian-test" \
  -e "container_function=dev" \
  -e "container_ip=10.92.3.201" \
  -e "container_template=local:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst"
```

### Privileged Container
```bash
ansible-playbook playbooks/deploy-proxmox-container.yml \
  -e "container_name=docker-host" \
  -e "container_function=utility" \
  -e "container_ip=10.92.3.185" \
  -e "container_privileged=true" \
  -e "container_memory=8192"
```

## Next Steps

After deployment:
1. SSH into the container/VM
2. Install application software
3. Configure application
4. Update NPM proxy settings if needed
5. Add to Prometheus targets (if custom metrics)
6. Test backup/restore

## Related Documentation

- `POSTGRESQL-HA-SETUP.md` - PostgreSQL HA configuration
- `SEMAPHORE-SETUP-INPUTS.md` - Semaphore integration
- `MSP-PLATFORM-ANALYSIS.md` - MSP platform architecture
