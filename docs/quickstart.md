# Quick Start

Deploy a VM for a new partner in 4 simple steps - takes 5 minutes.

## Prerequisites
- Git access to clone this template repository
- Ansible >=2.16, Python >=3.9
- VMware vCenter access
- VM template ready

## Steps

### 1. Clone Template for New Partner

First, clone this template repository and set it up for your new partner:

```bash
# Clone the template repository
git clone https://github.com/dustinliddick/vcenter_DeploymentTemplate.git vms_[PARTNER_NAME]
cd vms_[PARTNER_NAME]

# Remove the original git remote and create a new repository for this partner
git remote remove origin  # rename???
git remote add origin https://github.com/[YOUR_ORG]/vms_[PARTNER_NAME].git

# Make initial commit for the new partner repository
git add .
git commit -m "Initial setup for vms_[PARTNER_NAME] VM deployment"
git push -u origin main
```

### 2. Install
```bash
ansible-galaxy collection install <namespace>.vms_partner
pip install -r requirements.txt
```

### 3. Create playbook
Create a playbook file named after your target server (e.g., `partner-web-01.yml`):
```yaml
---
- name: Deploy Partner VM
  hosts: localhost
  connection: local
  gather_facts: false

  vars:
    vcenter_hostname: "vcenter.company.com"
    vcenter_username: "admin@vsphere.local" 
    vcenter_password: "password"  # Use ansible-vault for production
    vcenter_datacenter: "Datacenter1"
    vcenter_template: "RHEL9-Template"
    vcenter_esxi_host: "esxi01.company.com"

    vm_name: "partner-web-01"  # Match your playbook filename
    vm_networks:
      - name: "VM Network"
        ip: "192.168.1.100"
        netmask: "255.255.255.0"
        gateway: "192.168.1.1"

  roles:
    - dustinliddick.vms_partner.vmware_deploy
```

### 4. Deploy
```bash
ansible-playbook partner-web-01.yml
```

Done! Your VM is now running.

## Next Steps
- For detailed configuration options, see [single-vm-deployment.md](single-vm-deployment.md)
- Use `ansible-vault` for secure passwords
- Name playbooks after your target hostname (e.g., `web-server-01.yml`)
