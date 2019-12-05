Linux Sudo User Management
==========================

The playbook aims to create sudo users in linux. The playbook will prompt to enter the username and user will be assigned a default password and will be forced to change password on its first login.

---

Requirements
------------

***Template file*** should be available for configuring sudo command for user.

```yaml
{{ username }} ALL=(ALL) NOPASSWD: ALL
```

---

Role Variables
--------------

**Default_Variables**

| Name               | Default Value                      | Description                                                       |
| ------------------ | ---------------------------------- | ----------------------------------------------------------------- |
| default_password   | `{{ username }}`123                | The default password assigned to user                             |
| user_home          | `/home/{{ username }}`             | The home directory for the user                                   |
| user_group         | `docker`                           | The groups where the user should be assigned                      |
| user_shell         | `"/bin/bash"`                      | The default shell for the user                                    |

**Other_Variables**

| Name               |                                                                   |
|--------------------|-------------------------------------------------------------------|
| username           | Used to assign username for the user                              |
| setting_password   | Setting the password to be available to subsequent plays          |

---

Example 1 Playbook
------------------

```yaml
- name: "-----: ADDING USER :------"
  hosts: all
  become: yes
  connection: local
  vars_prompt:
   - name: username
     prompt: "Enter Username"
     private: no
  roles:
    - { role: users }
```
---

Example 2 Playbook
------------------

```yaml
- name: ADD User
  user:
       name: "{{ username }}"
       comment: "Sudo User"
       shell: "{{ user_shell }}"
       group: "{{ username }}"
       groups: "{{ user_group }}"
       createhome: True
       home: "{{ user_home }}"
       password: "{{ setting_password | string | password_hash('sha512', 'salty') }}"
       state: present
```
---

Usage
-----
After you clone this repo to your desktop, go to its root directory and run the playbook using `ansible-playbook deploy.yml` to run the playbook on hosts defined in `ansiserver file`.

If you want to check what are the changes playbook will make rather than making them, use `--check` with ansible-playbook to be execute.

Example: `ansible-playbook deploy.yml --check`

>You can check out the full description about `--check` [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html)

If you want to execute as root user, run the playbook using `sudo ansible-playbook deploy.yml`.

---

License
-------

MIT
