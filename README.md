# Nutanix Ansible Collection - nutanix.ncp
Official Nutanix ansible collections to automate Nutanix Cloud Platform (ncp).

# Installation

**Prerequisite**

Ansible should be pre-installed. If not, please follow official ansible [install guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) .

For <font color=royalblue>Developers</font>, please follow [this install guide](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html) for setting up dev environment.

**Clone the GitHub repository to a local directory**

```git clone https://github.com/nutanix/nutanix.ansible.git```

**Build the collection**

```ansible-galaxy collection build```

**Install the collection**

```ansible-galaxy collection install nutanix-ncp-<version>.tar.gz```

Add <font color=red>`--force`</font> option for rebuilding or reinstalling to overwrite existing data

# Included Content

## Modules

| Name | Description |
| --- | --- |
| ntnx_vms | Create or delete a VM. |
| ntnx_vpcs | Create or delete a VPC. |
| ntnx_subnets | Create or delete a Subnet. |
| ntnx_floating_ips | Create or delete a Floating Ip. |
| ntnx_pbrs | Create or delete a PBR. |

## Inventory Plugins

| Name | Description |
| --- | --- |
| ntnx_vms_inventory | Nutanix VMs inventory source |

# Module documentation and examples
```
ansible-doc nutanix.ncp.<module_name>
```

# How to contribute

We glady welcome contributions from the community. From updating the documentation to adding more functions for Ansible, all ideas are welcome. Thank you in advance for all of your issues, pull requests, and comments!

* [Contributing Guide](CONTRIBUTING.md)
* [Code of Conduct](CODE_OF_CONDUCT.md)

# Examples
## Playbook for IaaS provisioning on Nutanix

**Refer to `examples/iaas` for full implementation**

```yaml
---
- name: IaaS Provisioning
  hosts: localhost
  gather_facts: false
  collections:
    - nutanix.ncp
  vars:
      nutanix_host: <host_ip>
      nutanix_username: <user>
      nutanix_password: <pass>
      validate_certs: false
  ignore_errors: true
  tasks:
    - name: Inputs for external subnets task
      include_tasks: external_subnet.yml
      with_items:
        - { name: Ext-Nat, vlan_id: 102, ip: 10.44.3.192, prefix: 27, gip: 10.44.3.193, sip: 10.44.3.198, eip: 10.44.3.207, eNat: True }

    - name: Inputs for vpcs task
      include_tasks: vpc.yml
      with_items:
      - { name: Prod, subnet_name: Ext-Nat}
      - { name: Dev, subnet_name: Ext-Nat}

    - name: Inputs for overlay subnets
      include_tasks: overlay_subnet.yml
      with_items:
        - { name: Prod-SubnetA, vpc_name: Prod , nip: 10.1.1.0, prefix: 24, gip: 10.1.1.1, sip: 10.1.1.2, eip: 10.1.1.5,
            domain_name: "calm.nutanix.com", dns_servers : ["8.8.8.8","8.8.8.4"], domain_search: ["calm.nutanix.com","eng.nutanix.com"] }
        - { name: Prod-SubnetB, vpc_name: Prod , nip: 10.1.2.0, prefix: 24, gip: 10.1.2.1, sip: 10.1.2.2, eip: 10.1.2.5,
            domain_name: "calm.nutanix.com", dns_servers : ["8.8.8.8","8.8.8.4"], domain_search: ["calm.nutanix.com","eng.nutanix.com"] }
        - { name: Dev-SubnetA, vpc_name:  Dev , nip: 10.1.1.0, prefix: 24, gip: 10.1.1.1, sip: 10.1.1.2, eip: 10.1.1.5,
            domain_name: "calm.nutanix.com", dns_servers : ["8.8.8.8","8.8.8.4"], domain_search: ["calm.nutanix.com","eng.nutanix.com"] }
        - { name: Dev-SubnetB, vpc_name:  Dev , nip: 10.1.2.0, prefix: 24, gip: 10.1.2.1, sip: 10.1.2.2, eip: 10.1.2.5,
            domain_name: "calm.nutanix.com", dns_servers : ["8.8.8.8","8.8.8.4"], domain_search: ["calm.nutanix.com","eng.nutanix.com"] }

    - name: Inputs for vm task
      include_tasks: vm.yml
      with_items:
       - {name: "Prod-Wordpress-App", desc: "Prod-Wordpress-App", is_connected: True , subnet_name: Prod-SubnetA, image_name: "wordpress-appserver", private_ip: ""}
       - {name: "Prod-Wordpress-DB", desc: "Prod-Wordpress-DB", is_connected: True , subnet_name: Prod-SubnetB, image_name: "wordpress-db", private_ip: 10.1.2.5}
       - {name: "Dev-Wordpress-App", desc: "Dev-Wordpress-App", is_connected: True , subnet_name: Dev-SubnetA, image_name: "wordpress-appserver", private_ip: ""}
       - {name: "Dev-Wordpress-DB", desc: "Dev-Wordpress-DB", is_connected: True , subnet_name: Dev-SubnetB, image_name: "wordpress-db",private_ip: 10.1.2.5}

    - name: Inputs for Floating IP task
      include_tasks: fip.yml
      with_items:
        - {vm_name: "Prod-Wordpress-App"}
        - {vm_name: "Dev-Wordpress-App"}

```
