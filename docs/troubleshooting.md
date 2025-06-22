# Troubleshooting Single VM Deployments

Common issues and solutions when deploying single VMs with this collection.

## Quick Diagnosis

Run with verbose output to see what's happening:
```bash
ansible-playbook -vvv deploy-vm.yml
```

## Common Issues

### 1. VM Already Exists

**Error**: VM with name 'my-vm' already exists
**Solution**: This is normal! The collection is idempotent - it will skip VM creation but still apply configuration changes.

**To force recreation:**
```bash
# Delete the VM first
ansible-playbook playbooks/cleanup.yml -e vm_name=my-vm
# Then redeploy
ansible-playbook deploy-vm.yml
```

### 2. Network Configuration Fails

**Error**: Network 'VM Network' not found
**Solution**: Network names must exactly match vCenter port group names (case-sensitive).

**Check available networks:**
```yaml
- name: List Networks
  community.vmware.vmware_host_portgroup_info:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    esxi_hostname: "{{ vcenter_esxi_host }}"
    validate_certs: false
```

### 3. Template Not Found

**Error**: Template 'RHEL9-Template' not found in datacenter
**Solutions**:
- Verify template exists in vCenter
- Check template name spelling (case-sensitive)
- Ensure template is in the correct datacenter
- Check if template is a VM template, not regular VM

**List available templates:**
```bash
# Use vCenter web client or PowerCLI:
Get-Template -Location "Datacenter1"
```

### 4. Permission Denied

**Error**: User does not have permission to perform operation
**Required vCenter permissions**:
- Virtual machine.Provisioning.Deploy template
- Virtual machine.Configuration.Add new disk
- Virtual machine.Configuration.Change Settings
- Network.Assign network
- Datastore.Allocate space

**Quick fix**: Grant "Virtual Machine Administrator" role to your user.

### 5. ESXi Host Issues

**Error**: Host 'esxi01.company.com' not found
**Solutions**:
- Verify ESXi hostname is correct (FQDN)
- Check if host is connected and responsive
- Ensure sufficient resources (CPU, memory, storage)

**Check host status:**
```yaml
- name: Get ESXi Host Info
  community.vmware.vmware_host_info:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    esxi_hostname: "{{ vcenter_esxi_host }}"
    validate_certs: false
```

### 6. IP Assignment Timeout

**Error**: Timeout waiting for IP address
**Causes**:
- VM template doesn't have VMware Tools installed
- Network configuration conflicts
- DHCP vs static IP conflicts
- VM didn't boot properly

**Solutions**:
```yaml
# Increase timeout
vm_wait_timeout: 600  # 10 minutes

# Or skip IP wait
vm_wait_for_ip_address: false
```

### 7. Insufficient Resources

**Error**: Insufficient CPU/memory resources
**Solutions**:
- Check ESXi host resource availability
- Reduce VM resource requirements:
```yaml
memory_mb: 4096    # Reduce from default 16384
num_cpus: 2        # Reduce from default 4
```

### 8. SSL Certificate Issues

**Error**: SSL certificate verification failed
**Quick fix**: Disable certificate validation (development only):
```yaml
validate_certs: false
```

**Production fix**: Add vCenter certificate to trusted store or use proper CA-signed certificates.

### 9. Datastore Issues

**Error**: Insufficient disk space or datastore not accessible
**Solutions**:
- Check datastore capacity in vCenter
- Specify different datastore:
```yaml
vcenter_datastore: "SAN_Storage_01"
```
- Use thin provisioning for data disks:
```yaml
data_disk:
  type: thin
```

### 10. VM Customization Fails

**Error**: Guest customization failed
**Causes**:
- Template missing VMware Tools
- Guest OS customization issues
- Domain join problems

**Workaround**: Skip customization and configure manually:
```yaml
customize: false
```

## Debug Mode

Enable detailed logging:

```yaml
---
- name: Debug VM Deployment
  hosts: localhost
  connection: local
  gather_facts: false
  
  vars:
    # Your regular vars...
    
  tasks:
    - name: Deploy with debug info
      include_role:
        name: dustinliddick.vms_partner.vmware_deploy
      vars:
        debug: true
```

## Manual Verification Steps

### 1. Check VM Exists
```bash
# PowerCLI
Get-VM -Name "my-vm"

# vCenter web client
Browse to VMs and Templates > Your VM
```

### 2. Verify Network Configuration
```bash
# From VM console
ip addr show
ping gateway_ip
nslookup google.com
```

### 3. Check VMware Tools
```bash
# From VM
systemctl status vmtoolsd
vmware-toolbox-cmd stat hosttime
```

## Getting Help

### 1. Check VM Events in vCenter
vCenter > VMs and Templates > Your VM > Monitor > Events

### 2. Review ESXi Logs
```bash
# SSH to ESXi host
tail -f /var/log/vmware/vmkernel.log
tail -f /var/log/vmware/hostd.log
```

### 3. Ansible Verbose Output
```bash
# Maximum verbosity
ansible-playbook -vvvv deploy-vm.yml

# Show task timing
ansible-playbook --start-at-task="Clone VM" -vv deploy-vm.yml
```

### 4. Test Network Connectivity
```yaml
- name: Test vCenter Connection
  community.vmware.vmware_about_info:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: false
```

## Prevention Tips

1. **Test with minimal VM first**: Deploy with 1 CPU, 2GB RAM to verify basic functionality
2. **Use templates with VMware Tools**: Ensure your templates have VMware Tools pre-installed
3. **Verify network names**: Check port group names in vCenter before deployment
4. **Start with DHCP**: Test with DHCP first, then switch to static IPs
5. **Use ansible-vault**: Encrypt credentials from the start
6. **Check prerequisites**: Verify Python modules and Ansible version

## Recovery Commands

### Force VM Removal
```bash
ansible-playbook playbooks/cleanup.yml -e vm_name=problematic-vm
```

### Reset Failed Deployment
```bash
# Remove partial deployment
ansible-playbook playbooks/cleanup.yml -e vm_name=failed-vm
# Clean deploy
ansible-playbook deploy-vm.yml
```

### Manual VM Cleanup (if automation fails)
```bash
# PowerCLI
$vm = Get-VM "stuck-vm"
$vm | Stop-VM -Confirm:$false
$vm | Remove-VM -DeletePermanently -Confirm:$false
```

Remember: The collection is designed to be idempotent - you can safely re-run deployments to fix configuration issues.