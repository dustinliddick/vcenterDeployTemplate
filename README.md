# VMware Partner Template

A minimal Ansible template for rapid VMware vSphere VM provisioning in partner environments.

[![CI](https://github.com/dustinlidick/vcenter_DeploymentTemplate/workflows/CI/badge.svg)](https://github.com/dustinlidick/vcenter_DeplymentTemplate/actions)
[![Galaxy](https://img.shields.io/badge/galaxy-dustinlidick.vcenter_DeploymentTemplate-blue)](https://galaxy.ansible.com/dustinlidick/vcenter_DeploymentTemplate)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## Overview

This template repository provides a complete framework for deploying VMware VMs to partner environments. Clone this template, customize for your partner, and deploy VMs in minutes with idempotent, repeatable workflows.

## Features

- **Rapid VM cloning** - Deploy VMs in under 5 minutes
- **Idempotent playbooks** - Safe to run multiple times
- **Network configuration** - Static IP assignment with DNS
- **Optional data disks** - Attach additional storage as needed
- **Hardware customization** - CPU, memory, and disk resources
- **vCenter 8.0+ support** - Modern vSphere compatibility

## Getting Started

**Prerequisites:** Ansible >=2.16, Python >=3.9, VMware vCenter access

1. **Clone template for your partner:**
   ```bash
   git clone https://github.com/dustinliddick/vcenter_DeploymentTemplate.git partner-[NAME]
   cd partner-[NAME]
   ```

2. **Install dependencies:**
   ```bash
   ansible-galaxy collection install dustinliddick.vms_partner
   pip install -r requirements.txt
   ```

3. **Create and run playbook:**
   ```bash
   # Create partner-web-01.yml with your VM config
   ansible-playbook partner-web-01.yml
   ```

**Complete setup guide:** [docs/quickstart.md](docs/quickstart.md)

## Configuration

**Required variables:** vCenter credentials, datacenter, template, ESXi host, VM name, and network config.

**Optional:** CPU/memory sizing, additional disks, DNS settings, timeouts.

**Complete configuration guide:** [docs/single-vm-deployment.md](docs/single-vm-deployment.md)

## Architecture

```
.
├── docs/                   # Documentation
├── group_vars/             # Shared variables
├── host_vars/              # Per-host variables
├── playbooks/              # Example playbooks
├── roles/vmware_deploy/    # Main deployment role
└── requirements.txt        # Python dependencies
```

The `vmware_deploy` role handles VM cloning, network configuration, and hardware customization using the VMware vSphere API.

## Development

```bash
# Clone and setup
git clone https://github.com/dustinliddick/vcenter_DeploymentTemplate.git
cd vcenter_DeploymentTemplate
pip install -r requirements.txt

# Run tests
molecule test
ansible-lint
```

**Contributing:** Fork → Feature branch → Tests → PR

**Security:** Use `ansible-vault` for credentials, enable certificate validation in production.

## Release Information

**Status:** Production-ready
**Versioning:** [Semantic Versioning](https://semver.org/)
**Releases:** [GitHub Releases](https://github.com/dustinliddick/vcenter_DeploymentTemplate/releases)

## Roadmap

- [ ] Multi-datacenter support
- [ ] Windows VM templates
- [ ] Integration with HashiCorp Vault
- [ ] Terraform provider compatibility

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for detailed release history.

## Troubleshooting

**Common issues and solutions:** [docs/troubleshooting.md](docs/troubleshooting.md)

## License

[MIT License](LICENSE) - Free for commercial and personal use.

## Support

- **Issues:** [GitHub Issues](https://github.com/dustinliddick/vcenter_DeploymentTemplate/issues)
- **Discussions:** [GitHub Discussions](https://github.com/dustinliddick/vcenter_DeploymentTemplate/discussions)
- **Documentation:** [docs/](docs/)

## Acknowledgments

Built with [community.vmware](https://github.com/ansible-collections/community.vmware) and [pyvmomi](https://github.com/vmware/pyvmomi).

---

**Maintained by:** [Dustin Liddick](https://github.com/dustinliddick)
**Last Updated:** 2025-06-22
