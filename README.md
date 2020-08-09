# jenkins-ansible

This is a simple project following Infrastructure as Code on AWS. A default
CloudFormation template is provided which will outfit an AWS environment with
what is needed to SSH into an EC2 instance. The overall goal of this project is
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

#### Jenkins Server Creation

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
aws_ec2_ami: ami-098f16afa9edf40be
aws_ec2_instance_type: t2.micro
aws_ec2_instance_tag_name: jenkins-captain
aws_ec2_role_arn: arn:aws:iam::{{ aws_account_id }}:role/ansible-ec2-role
aws_ec2_pause_time: 2

aws_default_cloudformation_stack: true
aws_cloudformation_stack_name: ansible-jenkins-default
```

The credential section at the top should remain unchanged, but the variables
below may depend on your usage of the provided CloudFormation template.

A default `ansible.cfg` file is provided and has the following contents:

```
[defaults]
interpreter_python = auto_silent
host_key_checking = false
private_key_file = {{ full key path }}/keys/jenkins_aws_key.pem
```

Because this project utilizes a LinuxAcademy sandbox, it is not efficient to
store keys in a more traditional way. Instead, users should update the 
`private_key_file` variable with the path to their intended private key.

When ready to deploy, we will run the `provision.yml` playbook:

```
$ ansible-playbook provision --ask-vault-pass
```

When prompted, enter the password that was used during the encryption step.

NOTE: There has been some issues with Ansible accessing the EC2 instance, but
a `pause` has been added with the default value set to 2 minutes.

Once the playbook has completed, you will find an EC2 instance named
"jenkins-captain" configured with Jenkins.

At this time configuration has not been automated, but it will eventually be
included. Instead, please navigate to `http://{{ your_ec2_ip }}:8080` to access
your new Jenkins server.

To get the initial administrator password, SSH into your EC2 instance and run
the command:

```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### Clean up

When you're all done, tearing down the lab can be done by including `destroy` as
an extra variable:

```
$ ansible-playbook provision -e "destroy=true"
```

NOTE: During the stack deletion process there has been some instances of a hang 
when deleting the `ansiblePublicVpc` and that may need to be deleted manually.