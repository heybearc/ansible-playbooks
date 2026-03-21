# Semaphore Template Auto-Sync

This repository includes automation to automatically create Semaphore templates from playbooks.

## How It Works

When you add a new playbook to the `playbooks/` directory:

1. **Add playbook metadata** to `scripts/semaphore-auto-template.py`
2. **Run the sync playbook** in Semaphore
3. **New template is created automatically**

## Quick Sync

### Option 1: Via Semaphore UI (Recommended)

1. Go to https://ansible.cloudigan.net
2. Find template: **"Sync Semaphore Templates"**
3. Click **Run**
4. New templates are created automatically

### Option 2: Via Command Line

```bash
ssh ansible-control
export PATH=/usr/local/bin:$PATH
export LC_ALL=en_US.UTF-8

# Set credentials
export SEMAPHORE_PASSWORD="Cloudigan_Ansible_2026!"
export SEMAPHORE_PROJECT_ID="1"

# Run the sync playbook
ansible-playbook playbooks/sync-semaphore-templates.yml
```

### Option 3: Direct Script

```bash
cd scripts
export SEMAPHORE_PASSWORD="Cloudigan_Ansible_2026!"
export SEMAPHORE_PROJECT_ID="1"
python3 semaphore-auto-template.py
```

## Adding New Playbooks

### Step 1: Create Your Playbook

```bash
cd playbooks
vim my-new-feature.yml
```

### Step 2: Add Metadata

Edit `scripts/semaphore-auto-template.py` and add to `PLAYBOOK_METADATA`:

```python
"my-new-feature.yml": {
    "name": "My New Feature",
    "description": "What this playbook does",
    "allow_override_args": False
}
```

### Step 3: Commit to GitHub

```bash
git add playbooks/my-new-feature.yml scripts/semaphore-auto-template.py
git commit -m "feat: Add my-new-feature playbook"
git push
```

### Step 4: Sync Templates

Run the "Sync Semaphore Templates" task in Semaphore UI.

## Environment Variables

The sync process needs these variables:

| Variable | Default | Required | Description |
|----------|---------|----------|-------------|
| `SEMAPHORE_URL` | `https://ansible.cloudigan.net` | No | Semaphore URL |
| `SEMAPHORE_USER` | `admin` | No | Semaphore username |
| `SEMAPHORE_PASSWORD` | - | **Yes** | Semaphore password |
| `SEMAPHORE_PROJECT_ID` | `1` | No | Project ID |

### Setting in Semaphore

1. Go to **Environment** in Semaphore
2. Edit "Production" environment
3. Add environment variable:
   - Name: `SEMAPHORE_PASSWORD`
   - Value: `Cloudigan_Ansible_2026!`
   - Secret: ✅ (check this box)

## What Gets Created

For each playbook in `playbooks/` directory, a template is created with:

- **Name:** From metadata
- **Description:** From metadata
- **Playbook Path:** `playbooks/filename.yml`
- **Inventory:** Production Hosts
- **Repository:** Ansible Playbooks
- **Environment:** Production
- **Teams Alerts:** Enabled

## Scheduling Auto-Sync

You can schedule the sync to run automatically:

1. Edit "Sync Semaphore Templates" template
2. Enable **Schedule**
3. Set cron: `0 3 * * *` (daily at 3 AM)
4. Save

This ensures new playbooks are always available as templates.

## Troubleshooting

**Templates not created:**
- Check SEMAPHORE_PASSWORD is set correctly
- Verify playbook has metadata in the script
- Check Semaphore logs for errors

**Script fails:**
- Ensure `requests` and `PyYAML` are installed
- Verify network connectivity to Semaphore
- Check project ID is correct

**Playbook not found:**
- Ensure playbook is committed to GitHub
- Wait a few minutes for GitHub to update
- Check playbook is in `playbooks/` directory

## Current Playbooks

| Playbook | Template Name | Status |
|----------|---------------|--------|
| `fix-python-modules.yml` | Fix Python Modules | ✅ Active |
| `system-update.yml` | System Update | ✅ Active |
| `health-check.yml` | Health Check | ✅ Active |
| `nodejs-app-restart.yml` | Restart Node.js Apps | ✅ Active |
| `postgresql-status.yml` | PostgreSQL Status | ✅ Active |
| `sync-semaphore-templates.yml` | Sync Semaphore Templates | 🔄 Meta |

## Benefits

✅ **No manual template creation** - Add playbook, run sync, done  
✅ **Consistent configuration** - All templates use same settings  
✅ **Version controlled** - Playbook metadata in Git  
✅ **Self-documenting** - Script shows what will be created  
✅ **Idempotent** - Safe to run multiple times  

## Future Enhancements

- Auto-detect playbook metadata from YAML comments
- Webhook trigger on GitHub push
- Slack/Teams notification on new templates
- Template validation before creation
- Automatic scheduling configuration
