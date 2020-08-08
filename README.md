# jenkins-ansible

This is a simple project following Infrastructure as Code on AWS. A default
CloudFormation template is provided which will outfit an AWS environment with
what is needed to SSH into a EC2 instance. The overall goal of this project is
to eventually expand toward best practices in security and taking advantage of 
the ease of use with Ansible.

## Usage

### Environment set up

It is recommended that a virtual environment be set up, and dependencies can be
installed using the provided `requirements.txt` file:

```
$ python3 -m venv venv
$ source venv/bin/activate
$ pip install -r requirements.txt
```

A `vars/vault.yml` file should be created to store your AWS credentials, an
example of the contents is:

```
vault_aws_account_id: 000000000000
vault_aws_access_key_id: ABCDEFGHIJKLMNOPQRST
ault_aws_secret_access_key: 1234567890abcdefghijklmnopqrstuvwxyzABCD
vault_aws_ec2_key_name: your_key_pair_name_here
```

Once that file is created, we will then encrypt the file:

```
$ ansible-vault encrypt vars/vault.yml
```

Follow the prompts provided to secure your credentials.

### Example

This tool is built under the impression that a user has a clean slate account to
work with. The `default_ansible_jenkins.template` file will create many of the
preliminary resources and roles that will be required for the Ansible AWS 
modules to execute and provision EC2 instances. 

Updating user requirements can be done in the `aws.yml` file: 

```
---
aws_account_id: "{{ vault_aws_account_id }}"
aws_access_key_id: "{{ vault_aws_access_key_id }}"
aws_secret_access_key: "{{ vault_aws_secret_access_key }}"
aws_ec2_key_name: "{{ vault_aws_ec2_key_name }}"

aws_region: us-east-1
aws_ec2_security_group: JenkinsSG
aws_ec2_ami: ami-02354e95b39ca8dec
aws_ec2_instance_type: t2.micro
aws_ec2_instance_tag_name: jenkins-captain
aws_ec2_role_arn: arn:aws:iam::{{ aws_account_id }}:role/ansible-ec2-role

aws_default_cloudformation_stack: true
aws_cloudformation_stack_name: ansible-jenkins-default
```

The credential section at the top should remain unchanged, but the variables
below may depend on your usage of the provided CloudFormation template.

When ready to deploy, we will run the `provision.yml` playbook:

```
$ ansible-playbook provision --ask-vault-pass
```

When prompted, enter the password that was used during the encryption step.

Once the playbook has completed, you will find a barebones EC2 instance named
"jenkins-captain".