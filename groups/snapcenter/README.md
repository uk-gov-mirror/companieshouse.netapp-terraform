# NetApp Terraform | SnapCenter

This Terraform configuration deploys instances using the netapp-snapcenter-ami which includes:
- RHEL 9
- Required packages and dependencies
- Root user with temporary password - this is used and then removed in [netapp-snapcenter-ansible](https://github.com/companieshouse/netapp-snapcenter-ansible)
- NetApp SnapCenter server installed and enabled

- There are 2 EBS volumes:
  - **Root** - OS and system files
  - **Data** - Mounted at `/opt`, containing SnapCenter installation (inlcuding MySQL data)

## Monitoring

CloudWatch alarms are configured for:
- Instance status checks
- CPU utilisation
- Disk usage

## Required Post-run Ansible

Once a server has been provisioned with this ami, post-run ansible is required to apply environment-specific configurations, including securing the root account:
[netapp-snapcenter-ansible / 0-provision.yml](https://github.com/companieshouse/netapp-snapcenter-ansible#required-playbook)

## Optional: Destroying a Data Volume
To protect existing data volumes,`aws_ebs_volume.snapcenter_data` is set to  `prevent_destroy = true`

If a full rebuild is ever required:
1. Temporarily remove `prevent_destroy` from the `lifecycle` block in `instance.tf`
2. `terraform-runner -g snapcenter -p [environment]-eu-west-2 -e [env-type] -c destroy`
3. Revert `instance.tf` and confirm a clean `plan` before merging

Note: This change will be shared across profiles, so protection will be off during this process; ensure you only target the intended environment(s) in step 2.

______________________  

# SnapCenter Overview

SnapCenter is NetApp's data protection software that provides backup, restore, and clone capabilities for various applications and databases.


SnapCenter 6.1+ on Linux requires:
- RHEL 8 or 9
- Minimum 4 cores, 8GB RAM
- 15GB disk space for SnapCenter Server and repository
- .NET Framework 8.0.12+
- PowerShell 7.4.2+
- Nginx (reverse proxy)


| Port | Protocol | Description |
|------|----------|-------------|
| 22 | SSH | Administrative access |
| 3306 | TCP | MySQL repository |
| 5672 | TCP | RabbitMQ messaging |
| 8145 | HTTPS | SMCore communication |
| 8146 | HTTPS | SnapCenter main port (Web UI & API) |
| 8154 | HTTPS | Scheduler service |
