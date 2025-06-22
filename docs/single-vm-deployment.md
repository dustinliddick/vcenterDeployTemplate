# Single VM Deployment Guide

Complete guide for deploying and configuring single VMs using the VMware Partner Template collection.

> **Quick Start**: For the fastest deployment, see [quickstart.md](quickstart.md) - this guide covers comprehensive configuration options.

## Single VM Configuration Options

### Required Variables
```yaml
vcenter_hostname: "vcenter.company.com"    # Your vCenter server
vcenter_username: "admin@vsphere.local"    # vCenter username
vcenter_password: "password123"            # vCenter password
vcenter_datacenter: "Datacenter1"          # Target datacenter
vcenter_template: "RHEL9-Template"         # VM template to clone
vcenter_esxi_host: "esxi01.company.com"    # Target ESXi host
vm_name: "app-server-prod-01"               # Name for new VM (use actual hostname)

vm_networks:                               # Network configuration
  - name: "VM Network"
    ip: "192.168.1.100"
    netmask: "255.255.255.0"
    gateway: "192.168.1.1"
```

### Optional Customizations
```yaml
# Hardware (defaults shown)
memory_mb: 16384        # RAM in MB (default: 16GB)
num_cpus: 4            # CPU count (default: 4)

# Domain settings
vcenter_guest_domain: "company.local"    # Guest OS domain

# VM placement
vcenter_folder: "/vm"   # vCenter folder path

# Additional storage
add_data_disk: true
data_disk:
  size_gb: 100         # Data disk size
  type: thin           # Provisioning type (thin/thick)

# Timeouts
vm_wait_timeout: 300   # Seconds to wait for IP assignment
```

## Multiple Network Interfaces

For VMs with multiple NICs:

```yaml
vm_networks:
  - name: "Production_Network"
    ip: "10.0.1.100"
    netmask: "255.255.255.0"
    gateway: "10.0.1.1"
  - name: "Management_Network"
    ip: "192.168.1.100"
    netmask: "255.255.255.0"
    gateway: "192.168.1.1"
```

## Using Variable Files (Recommended)

Instead of hardcoding variables in playbooks, use variable files:

1. **Create a variable file in the `vars/` directory named after your target server. For example, for server `app-pdm-mdle-1p`, create `vars/VARS_app-pdm-mdle-1p.yml` in the project root:**
   ```yaml
   vcenter_hostname: vcenter.company.com
   vcenter_username: "{{ vault_vcenter_user }}"
   vcenter_password: "{{ vault_vcenter_password }}"
   vcenter_datacenter: Production_DC
   vcenter_template: RHEL9-Gold
   vcenter_esxi_host: esxi-prod-01.company.com

   vm_name: app-pdm-mdle-1p
   memory_mb: 8192
   num_cpus: 2

   vm_networks:
     - name: App_Network
       ip: 10.20.30.100
       netmask: 255.255.255.0
       gateway: 10.20.30.1
   ```

2. **Create a simple playbook in the project root directory (e.g., `app-pdm-mdle-1p.yml`):**
   ```yaml
   ---
   - name: Deploy Single VM
     hosts: localhost
     connection: local
     gather_facts: false
     vars_files:
       - vars/VARS_app-pdm-mdle-1p.yml

     roles:
       - dustinliddick.vms_partner.vmware_deploy
   ```

3. **Deploy using the server-named playbook file:**
   ```bash
   ansible-playbook app-pdm-mdle-1p.yml
   ```

## Command Line Variable Override

Deploy with command-line customizations using your server-named playbook:

```bash
ansible-playbook web-server-prod-02.yml \
  -e vm_name=web-server-prod-02 \
  -e vm_networks='[{"name":"Web_Network","ip":"10.0.2.50","netmask":"255.255.255.0","gateway":"10.0.2.1"}]' \
  -e memory_mb=32768 \
  -e num_cpus=8
```

## Securing Credentials

Use Ansible Vault for passwords:

1. **Create encrypted variable file in the project root directory:**
   ```bash
   ansible-vault create vault.yml
   ```

2. **Add credentials:**
   ```yaml
   vault_vcenter_user: administrator@vsphere.local
   vault_vcenter_password: your-secure-password
   ```

3. **Reference in playbook:**
   ```yaml
   vars_files:
     - vault.yml
   vars:
     vcenter_username: "{{ vault_vcenter_user }}"
     vcenter_password: "{{ vault_vcenter_password }}"
   ```

4. **Run with vault password using your server-named playbook:**
   ```bash
   ansible-playbook app-pdm-mdle-1p.yml --ask-vault-pass
   ```

## VM with Additional Disk

Deploy a VM with extra storage:

```yaml
---
- name: Deploy VM with Data Disk
  hosts: localhost
  connection: local
  gather_facts: false

  vars:
    # ... standard variables ...
    add_data_disk: true
    data_disk:
      size_gb: 500
      type: thin

  roles:
    - dustinliddick.vms_partner.vmware_deploy
```

## What Happens During Deployment

The deployment process:

1. **Checks if VM exists** - Skips if already deployed (idempotent)
2. **Clones from template** - Creates new VM from specified template
3. **Configures hardware** - Sets CPU, memory, and disk resources
4. **Sets up networking** - Configures static IP addresses
5. **Applies customization** - Sets hostname, domain, DNS
6. **Powers on VM** - Starts the virtual machine
7. **Waits for IP** - Confirms network connectivity
8. **Reports success** - Provides VM details

## Verifying Deployment

Check if your VM deployed successfully:

```bash
# Basic verification playbook
---
- name: Verify VM Deployment
  hosts: localhost
  connection: local
  gather_facts: false

  tasks:
    - name: Get VM info
      community.vmware.vmware_guest_info:
        hostname: "{{ vcenter_hostname }}"
        username: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
        datacenter: "{{ vcenter_datacenter }}"
        name: "{{ vm_name }}"
        validate_certs: false
      register: vm_info

    - name: Display VM status
      debug:
        msg: "VM {{ vm_name }} is {{ vm_info.instance.hw_power_status }} with IP {{ vm_info.instance.hw_eth0.ipaddresses[0] }}"
```

## Common Single VM Use Cases

### Development Server
```yaml
vm_name: dev-app-test-01
memory_mb: 8192
num_cpus: 4
vm_networks:
  - name: Dev_Network
    ip: 192.168.10.100
    netmask: 255.255.255.0
    gateway: 192.168.10.1
```

### Database Server with Extra Storage
```yaml
vm_name: db-mysql-prod-01
memory_mb: 32768
num_cpus: 8
add_data_disk: true
data_disk:
  size_gb: 1000
  type: thick
vm_networks:
  - name: Database_Network
    ip: 10.0.100.50
    netmask: 255.255.255.0
    gateway: 10.0.100.1
```

### Web Server with Dual NICs
```yaml
vm_name: web-nginx-dmz-01
memory_mb: 16384
num_cpus: 6
vm_networks:
  - name: Web_DMZ
    ip: 203.0.113.100
    netmask: 255.255.255.240
    gateway: 203.0.113.97
  - name: Internal_Network
    ip: 10.0.1.100
    netmask: 255.255.255.0
    gateway: 10.0.1.1
```

This collection is specifically designed for single VM deployments - use these examples as your starting point and customize as needed for your environment.

