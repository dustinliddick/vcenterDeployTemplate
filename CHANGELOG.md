# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release of the collection
- vmware_deploy role for VM provisioning
- Comprehensive documentation and examples
- GitHub Actions CI/CD pipeline
- Molecule testing framework
- Python requirements specification

### Changed
- Refactored from dustinliddick.vcenter_DeploymentTemplate to collegis.vms_partner
- Updated namespace and collection naming
- Improved variable contracts and documentation

### Fixed
- Idempotence issues in VM creation task
- Variable validation and error handling
- Template placeholders removed from production code

## [1.0.0] - 2025-06-22

### Added
- vmware_deploy role for automated VM provisioning
- Support for VMware vSphere 8.0+
- Template-based VM cloning with customization
- Network configuration with static IP assignment
- Optional data disk attachment
- Hardware resource configuration (CPU, memory)
- Comprehensive variable validation
- Molecule testing with Docker
- GitHub Actions CI pipeline
- Complete documentation and examples

### Features
- **Idempotent Operations**: Safe to run multiple times
- **Template Cloning**: Clone VMs from existing templates
- **Network Configuration**: Configure multiple NICs with static IPs
- **Hardware Customization**: Set CPU, memory, and disk resources
- **Data Disk Support**: Optional additional storage attachment
- **DNS Integration**: Configure guest OS DNS settings
- **Comprehensive Testing**: Unit and integration tests
- **CI/CD Ready**: GitHub Actions workflow included

### Requirements
- Ansible >= 2.16
- Python >= 3.9
- pyvmomi == 8.0.2
- community.vmware >= 3.7.0, < 4.0.0

### Security
- Credential management via ansible-vault
- SSL certificate validation options
- Principle of least privilege recommendations

---

**Note**: This project follows semantic versioning. For upgrade instructions and breaking changes, see the README.md file.