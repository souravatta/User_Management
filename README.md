Linux Sudo User Management
==========================

The playbook aims to create sudo users in linux. The playbook will prompt to enter the username and the groups you want to add the user (default group is user's name) and user will be assigned a default password and will be forced to change password on its first login.

---

Requirements
------------
1) There should be ssh connection between the master node and all other host nodes.
2) ***Template file*** should be available for configuring sudo command for user.

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
| user_groups        | Used to assign group to user                                      |
| setting_password   | Setting the password to be available to subsequent plays          |

---

Example 1 Playbook
------------------

```yaml
- name: "----: ADDING USER :-----"
  hosts: all
  vars_prompt:
   - name: username
     prompt: "Enter Username"
     private: no
   - name: user_groups
     prompt: "Enter the groups name (separated by comma)"
     private: no
  roles:
    - { role: users }
```
---

Example 2 Playbook
------------------

```yaml
- name: Create group first
  group:
    name: "{{ username }}"
    state: present
  become: yes
  become_method: sudo

- name: ADD User
  user:
       name: "{{ username }}"
       shell: "{{ user_shell }}"
       group: "{{ username }}"
       groups: "{{ user_groups }}"
       createhome: True
       home: "{{ user_home }}"
       password: "{{ setting_password | string | password_hash('sha512', 'salty') }}"
       state: present
  become: yes
  become_method: sudo

- name: Change ownership of home directory
  file:
    path: "{{ user_home }}"
    owner: "{{ username }}"
    group: "{{ username }}"
    state: directory
  become: yes
  become_method: sudo
```
---

Usage
-----
1) After you clone this repo to your desktop, go to its root directory and change the files **ansible.cfg** and **ansiserver**.
2) In file **ansiserver**, add the hosts you want to run the playbook. 
3) In file **ansible.cfg** change the path of inventory file.
4) If the ssh user used to login hosts doesn't have ssh password, then run the playbook using `ansible-playbook deploy.yml`.
5) If the ssh user used to login hosts have ssh password, then run the playbook using `ansible-playbook deploy.yml -k`. It will prompt for `ssh password`.

If you want to check what are the changes playbook will make rather than making them, use `--check` with ansible-playbook to be execute.

Example: `ansible-playbook deploy.yml --check`

>You can check out the full description about `--check` [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html)

---

License
-------

MIT
