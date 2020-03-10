# ansible-role-aws-pci-instances

Provision AWS resources to support a PCI compliant HA web application:
  - An ssh key pair in AWS so you can provision Ec2 instances (assumes you have ~/.ssh/id_rsa.pub on your local machine)
  - EC2 security groups to enable communication between the various services
  - A bastion host EC2 instance with a public, permanent IP address, since the rest of the systems are on a private network with no public IPs
  - An RDS instance on the private network
  - An Elasticache instance on the private network
  - EC2 instances for application and NFS nodes on the private network
  - An application load balancer that redirects HTTP to HTTPS by default, and forwards HTTPS requests to the app nodes

## Dependencies / Requirements

- A hosted zone in Route 53. Set it up manually first. A subdomain works fine if you set up delegation to route53 for it.
- An ACM SSL certificate registered for the laod balancer to use. Set it up manually first. Specify its ARN in your group_vars.
- Ansible 2.9.1 or better. Run Ansible from source if your distro version isn't new enough. Ansible v2.9.0 seems to have an issue with timeouts during security group creation.
- AWS CLI
- Python pip modules:
  - boto
  - boto3
- Ansible role: [acromedia.aws-pci-network](https://github.com/AcroMedia/ansible-role-aws-pci-network)

## Required Variables

- **udf_template_path** - This needs to come from your local playbook. Its the script that gets passed to and executed on new EC2 instances
- **everything in defaults/main.yml** Copy defaults/main.yml to your own group_vars/all.yml filea as a starting point. Everything in there is required.
- **variables registered by acromedia.ansible-role-aws-pci-network** - Running this role independently wont work.

## Example playbook
**group_vars/all.yml**:
```yaml
# Copy everything from defaults/main.yml to your own group_vars/all.yml as a starting point.
```

**requirements.yml**:
```yaml
---
- name: acromedia.aws-pci-network
  src: https://github.com/AcroMedia/ansible-role-aws-pci-network
  version: origin/master

- name: acromedia.aws-pci-instances
  src: https://github.com/AcroMedia/ansible-role-aws-pci-instances.git
  version: origin/master
```

**playbook.yml**:
```yaml
- name: Provision AWS resources
  hosts: localhost
  connection: local
  become: false
  gather_facts: false
  tags:
    - vpc
  roles:
    - name: Configure VPC, subnets, and routing
      role: contrib/acromedia.aws-pci-network    # See group_vars/all.yml

    - name: Provision EC2, RDS, Elasticache, app nodes, load balancer
      role: contrib/acromedia.aws-pci-instances  # See group_vars/all.yml
```
