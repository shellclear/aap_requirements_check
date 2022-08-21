Ansible Role - aap_requirements_check
====================================

This role was designed to check the AAP node requirements before to run the AAP installer.

Limitations
-----------

 * Does not support Ansible Check Mode (Dry Run).

Requirements
------------

Ansible: ansible-core 2.13 or higher
Python: 3.8 or later 

To use correctly this role you will need to create an inventory file with the structure below and define the target nodes inside the groups according its function:

```command
@all:
  |--@automationcontroller:
  |--@automationhub:
  |--@database:
  |--@execution_nodes:
  |--@ungrouped:
```

Create the corresponding group_vars directories where you will create the variable files. I suggest the following structure of folder and files below:

```command
group_vars/
├── automationcontroller
│   ├── compute.yml
│   └── storage.yml
├── automationhub
│   ├── compute.yml
│   └── storage.yml
├── database
│   ├── compute.yml
│   └── storage.yml
└── execution_nodes
    ├── compute.yml
    └── storage.yml
```

groups_vars/automationcontroller/compute.yml sample file:

```command
cat << eof > compute.yml
vcpu: 2
memory: 2048
eof
```
groups_vars/automationcontroller/storage.yml sample file:

```command
cat << eof > storage.yml
volumes:
  - mountpoint: /
    size: 10 # The value in GB
eof
```

NOTE: You must have access to a privileged account on the target nodes to run these tasks.

Role Variables
--------------

| Name                                 | Default value                                           |                                                        |
|--------------------------------------|---------------------------------------------------------|--------------------------------------------------------|
| redhat_version                       | 8.3                                                     | Specify minimum required Red Hat OS version.           |
| cpu                                  | MANDATORY                                               | Specify cpu number                                     |
| memory                               | MANDATORY                                               | Specify memory number                                  |
| volumes                              | MANDATORY                                               | Specify mountpoint volume and size                     |
| ansible_username                     | ansible                                                 | Specify the ansible user on the target nodes           |
| ansible_user_password                | MANDATORY                                               | Specify the ansible user password                      |
| authorized_ssh_pub_key_ansible_user  | UNDEF                                                   | Specify SSH public key deployed on the target nodes    | 
| rhn_username                         | MANDATORY                                               | RHN username to connect Red Hat Cloud services         |
| rhn_password                         | MANDATORY                                               | RHN password to connect Red Hat Cloud services         |
| pool_ids                             | OMIT                                                    | Number of pool(users with SCA enabled dont need it)    |
| rhn_force_register                   | true                                                    | If defined, force re-register of machines              |
| ansible_repository                   | ansible-automation-platform-2.1-for-rhel-8-x86_64-rpms  | Ansible packages repository                            |
| common_packets                       | [chrony,firewalld]                                      | Specify default packages that need to be installed     |
| extra_packets                        | UNDEF                                                   | If defined, install the additional packages to install |
| common_services                      | [chrony,firewalld]                                      | Specify the basic services to install                  |
| common_ntp_pool                      | 2.rhel.pool.ntp.org                                     | Specify a NTP server                                   |
| timezone                             | MANDATORY                                               | Specify a timezone                                     |
| pg_port                              | 5432                                                    | Specify postgresql database port to open               | 

Example Playbook
----------------
playbook.yml

```yaml
cat << eof > playbook.yml
---
- name: Example playbook
  hosts: all
  roles:
    - aap_requirements_check
eof
```

Example Secret File
-------------------
secrets.yml

Run the command below to create a Ansible vault file:

```command
$ ansible-vault create secrets.yml
```
Create the variables, for instance:

```command
ansible_user_password: 4ns1bl3
rhn_password: YOUR_RHN_USER_PASSWORD 
```

Example Extra Vars File
-----------------------
extraVars.yml

```command
cat << eof > extraVars.yml
rhn_username: YOUR_RHN_USER@redhat.com
timezone: Europep/Madrid
eof
```

Example Ansible Playbook Command:
---------------------------------
I'm skipping some tasks to make it more quickly

```command
ANSIBLE_NOCOWS=true ansible-playbook -k -i inventory -e @secrets.yml --ask-vault-pass -e @extraVars.yml --skip-tags subscription,install_packages,configure_chrony,configure_database_firewall_rules,services playbook.yml
```

TAGS Available:

```command
playbook: playbook.yml
  play #1 (all): Ensure the nodes has the compute, storage and basic configuration requirements TAGS: []
      TASK TAGS: [ansible_credentials, compute_storage, configure_chrony, configure_database_firewall_rules, install_packages, os_version, services, subscription]
```

[![asciicast](https://asciinema.org/a/516079.svg)](https://asciinema.org/a/516079)

License
-------

BSD

Author Information
------------------

shellclear@gmail.com