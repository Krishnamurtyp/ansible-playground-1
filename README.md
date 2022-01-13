# ansible-playground

## Learning path

- https://github.com/ansible/ansible-examples
- - https://docs.ansible.com/ansible/2.8/user_guide/playbooks_reuse_roles.html

## First steps

It looks like you can use ansible playbooks by themselves in combination with a file that defines the hosts the playbook should target.

You'll want to begin by making sure you have your ssh keys in the ~/.ssh/authorized_keys file on your linux hosts you intend to manage with Ansible. It's also a good idea to make sure that user is set up with passwordless sudo. If you don't, you'll have to manage your secrets some other way. The basic 'cloud' pattern seems to be ssh key-only authentication and no password at all. In those cases especially, you'll want to make sure to use `visudo` to edit your sudoers file so you don't break it and lock yourself out of your own machine.

`/etc/ansible/hosts`: 
```ini
[host-group-name]
host1.example.com

[second-groupname]
host2.example.com
```

Example Playbooks to sync my preferred dotfiles:

`~/playbooks/yadm.yml`:
```yml
---
- name: "checkout yadm"
  hosts: all

  tasks:
    - name: Update repositories cache and install "git" package
      become: yes
      become_method: sudo
      apt:
        name: git
        update_cache: no

    - name: install "yadm"
      become: yes
      become_method: sudo
      apt:
        name: yadm

    - name: "Ansible | Print 'lsb_release"
      debug:
        msg: "{{ ansible_distribution_release }}"

    - name: checkout yadm repo [focal]
      command: yadm clone https://github.com/aaroneg/yadm-files.git
      args:
        creates: ~/.config/yadm/repo.git
      become: no
      when: ansible_facts['distribution_release']=='focal'

    - name: checkout yadm repo (root) [focal]
      command: yadm clone https://github.com/aaroneg/yadm-files.git
      args:
        creates: ~/.config/yadm/repo.git
      become: yes
      become_method: sudo
      when: ansible_facts['distribution_release']=='focal'

    - name: checkout yadm repo [bionic]
      command: yadm clone https://github.com/aaroneg/yadm-files.git
      args:
        creates: ~/.yadm/
      become: no
      when: ansible_facts['distribution_release']=='bionic'

```

`~/playbooks/yadm-pull.yml`:
```yml
---
- name: update yadm files
  hosts: all
  gather_facts: no
  tasks:

  - name: update yadm
    command: yadm pull
    become: no

  - name: update yadm (root)
    command: yadm pull
    become: yes
    become_method: sudo
```

Assuming you have the `hosts:` line correctly defined to target the hosts you want, it's a simple as:

`ansible-playbook yadm.yml`

## Beyond the basics

### Host Inventory

Ansible [_really_ prefers](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ini_inventory.html#notes) that you don't use the ini format for your inventory. For the next steps we'll convert our hosts to a yaml based file, even though I really hate yaml.

```yml
all:
    hosts:
        host1.example.com:
    children:
        apache:
            hosts:
                web1.example.com:
                web2.example.com:
        nginx:
            hosts:
                web3.example.com:
                web4.example.com:
        webservers:
            children:
                apache:
                nginx:
```

In this example, "all" contains:

- host1.example.com
- web1.example.com
- web2.example.com
- web3.example.com
- web4.example.com

The group - "webservers" contains all apache and nginx hosts, but not `host1.example.com`. In your environment you may wish to organize hosts in some other way, and there's no real rulebook for it, it's just whatever works best for your environment, and you can have more than one inventory file if needed. You can also use [patterns](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#adding-ranges-of-hosts) in host file entries, for cases where you have a large number of host names that only differ by a number or letter pattern somewhere in the hostname.


