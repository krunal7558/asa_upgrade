ASA upgrade Playbooks
=====================

Set of playbooks to automate ASA software upgrade.


Notable files and directories:

* hosts: Ansible inventory file. List of ASA firewalls hostnname and IP addresses in ansible_host variable. 

* ansible.cfg: Ansible configuration file. I had set connect_timeout = 300 so file trasnfer can take upto 5mins only. 

* group_vars/*: Group specific variable files.

* templates/*: TextFSM templates for ASA Stolen from ntc-templates. 

* vars/*: Generic variable files. Currently empty

* backup/*: backup of running configs.

* vault/*: Encrypted files created using ansible-vault. 

## Steps to create vault file to store username and password.

Create vault file which holds all secret passwords e.g TACACS passwords and enable passwords

	/asa_upgrade$ mkdir vault
	/asa_upgrade$ cd vault
	/asa_upgrade$ ansible-vault create secret.yml
	New Vault password:
	Confirm New Vault password:

Remember this password. Inside secret.yml add following and replace with your own username password

	---
	ansible_user: myusername # admin
	ansible_ssh_pass: topsecret # or admin
	enable_pass: everyoneknows # whocares
	# Ansible 2.5
	ansible_password: topsecret
	ansible_become_pass: everyoneknows

Save and exit this file can be edited only using "ansible-vault edit secret.yml" command

Create vault password file vault_pass.txt and assign appropriate permission. 

	/asa_upgrade $ vim .vault_pass.txt

Vim editor will open, go to insert mode and type your vault password then save and exit ESC + !wq

Change file mode so only you can read it

	/asa_upgrade $ chmod 400 .vault_pass.txt

## Playbooks:

```asa_facts.yml```: Playbook to collect hostname, software and hardware info and create CSV file. It also backs up running configuration. This playbook can be run against same firewall which 
needs to be upgraded as pre and post checks.
 
```/asa_upgrade$ ansible-playbook asa_facts.yml```


```asa_upgrade.yml```: Playbooks to copy new image to ASA(this casn be done using SCP and TFTP) , verify checksum, set new bootvar and reload the firewall. ```serial: 5``` parameter indicates 
how many firewall can be upgraded at any given time to minimize impact. 

```/asa_upgrade$ ansible-playbook asa_upgrade.yml```

