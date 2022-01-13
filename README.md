# ansible-playground

## Learning path

- https://github.com/ansible/ansible-examples
- - https://docs.ansible.com/ansible/2.8/user_guide/playbooks_reuse_roles.html

## First steps

It looks like you can use ansible playbooks by themselves in combination with a file that defines the hosts the playbook should target.

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

  - name: checkout yadm repo [xenial]
    command: yadm clone https://github.com/aaroneg/yadm-files.git
    args:
      creates: ~/.config/yadm/repo.git
    become: no
    when: ansible_facts['lsb']['codename']=='xenial'

  - name: checkout yadm repo (root) [xenial]
    command: yadm clone https://github.com/aaroneg/yadm-files.git
    args:
      creates: ~/.config/yadm/repo.git
    become: yes
    become_method: sudo
    when: ansible_facts['lsb']['codename']=='xenial'

  - name: checkout yadm repo [bionic]
    command: yadm clone https://github.com/aaroneg/yadm-files.git
    args:
      creates: ~/.yadm/repo.git
    become: no
    when: ansible_facts['lsb']['codename']=='bionic'
```

`~/playbooks/yadm-pull.yml`:
```yml
---
- name: update yadm files
  hosts: all

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

