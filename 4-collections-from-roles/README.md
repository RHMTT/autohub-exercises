# Exercise 4 - Running Collections from Roles

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
    - [Step 1: Understand collections lookup](#step-1-understand-collections-lookup)
    - [Step 2: Running collections from a role](step-2-running-collections-from-a-role)
- [Takeaways](#takeaways)

# Objective

This exercise will help users understand how collections are used from within roles.

Covered topics:

- Explain collection resolution logic

- Demonstrate how to call collections from roles using collection fully qualified collection name (FQCN)

# Guide

## Step 1: Understand collections lookup

Ansible Collections use a simple method to define collection namespaces. Anyway, if
your playbook loads collections using the `collections` key and one or more roles, then
the roles will not inherit the collections set by the playbook.

This leads to the main topic of this exercise: roles have an independent collection
loading method based on the role's metadata.
To control collections search for the tasks inside the role, users can choose between
two approaches:

- **Approach 1**: Pass a list of collections in the `collections` field inside the `meta/main.yml` file within the role.
  This will ensure that the collections list searched by the role will have higher priority than
  the collections list in the playbook. Ansible will use the collections list defined inside the role
  even if the playbook that calls the role defines different collections in a separate `collections` keyword entry.

  ```yaml
  # myrole/meta/main.yml
  collections:
    - my_namespace.first_collection
    - my_namespace.second_collection
    - other_namespace.other_collection
  ```

- **Approach 2**: Use the collection fully qualified collection name (FQCN) directly from a task in the role.
  In this way the collection will always be called with its unique FQCN, and override any other
  lookup in the playbook

  ```yaml
  - name: Create an EC2 instance using collection by FQCN
    amazon.aws.ec2:
      key_name: mykey
      instance_type: t2.micro
      image: ami-123456
      wait: yes
      group: webserver
      count: 3
      vpc_subnet_id: subnet-29e63245
      assign_public_ip: yes
  ```

Roles defined **within** a collection always implicitly search their own collection
first, so there is no need to use the `collections` keyword in the role metadata to access modules, plugins, or other roles.

## Step 2: Running collections from a role

Start from our working directory
```bash
cd ~/workspace/ansible_collections
```

For this lab, we will use the `ansible.posix` & `workshop.demo_collection` collections.

```bash
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install workshop.demo_collection
```

### Approach 1: Collections loaded as metadata

> **TIP**: You can go though the exercise steps or copy the finished role from `solutions/demo_role_meta`.

Create a new role using the `ansible-galaxy init` command:

```bash
ansible-galaxy init --init-path roles demo_role
```

Edit the `roles/demo_role/meta/main.yml` file and append the following lines at the end of the file, beginning at column 1:

```yaml
# Collections list
collections:
  - ansible.posix
  - workshop.demo_collection
```

Edit the `roles/demo_role/tasks/main.yml` file and add the following tasks:

```yaml
---
# tasks file for selinux_manage_meta
- name: Enable SELinux enforcing mode
  selinux:
    policy: targeted
    state: "{{ selinux_mode }}"

- name: Enable booleans
  seboolean:
    name: "{{ item }}"
    state: true
    persistent: true
  loop: "{{ sebooleans_enable }}"

- name: Disable booleans
  seboolean:
    name: "{{ item }}"
    state: false
    persistent: true
  loop: "{{ sebooleans_disable }}"

- name: Generate greeting and store result
  demo_hello:
    name: "{{ friend_name }}"
  register: demo_greeting

- name: Show value from demo_hello
  debug:
    var: demo_greeting
```

> **NOTE:** We're using the simple module name. Ansible uses the information from the
> `collections` list in the metadata file to locate the collection(s) used.

Edit the `roles/demo_role/defaults/main.yml` to define default values for role variables:

```yaml
---
# defaults file for selinux_manage_meta
selinux_mode: enforcing
sebooleans_enable: []
sebooleans_disable: []
friend_name: Jane Doe
```

Cleanup unused folders in the role:

```bash
rm -rf roles/demo_role/{tests,vars,handlers,files,templates}
```

We can now test the new role with a basic playbook. Create the `role_playbook.yml` file

~/workspace/ansible_collections/role_playbook.yml


```yaml
---
- hosts: localhost
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
    sebooleans_enable:
      - httpd_can_network_connect
      - httpd_mod_auth_pam
    sebooleans_disable:
      - httpd_enable_cgi
  roles:
    - demo_role
```

Run the playbook and check the results:


### Approach 2: Collections loaded with FQCN

The second approach uses the collection FQCN to call the related modules and plugins. To better demonstrate the differences, we will implement a new version of the previous role with the FQCN approach without changing the inner logic.

> **TIP**: You can go though the exercise steps or copy the finished role from `solutions/demo_role_fqcn`.

Create a new role using the `ansible-galaxy init` command:

```bash
ansible-galaxy init --init-path roles demo_role_fqcn
```

Edit the `roles/demo_role_fqcn/tasks/main.yml` file and add the following tasks:

```yaml
---
# tasks file for selinux_manage_fqcn
- name: Enable SELinux enforcing mode
  ansible.posix.selinux:
    policy: targeted
    state: "{{ selinux_mode }}"

- name: Enable booleans
  ansible.posix.seboolean:
    name: "{{ item }}"
    state: true
    persistent: true
  loop: "{{ sebooleans_enable }}"

- name: Disable booleans
  ansible.posix.seboolean:
    name: "{{ item }}"
    state: false
    persistent: true
  loop: "{{ sebooleans_disable }}"
```

> **NOTE**: Notice the usage of the modules FQCN in the role tasks. Ansible will directly
> look for the installed collection from within the role task, no matter if
> the `collections` keyword is defined at playbook level.

Edit the `roles/demo_role_fqcn/defaults/main.yml` to define default values for role variables:

```yaml
---
# defaults file for selinux_manage_fqcn
selinux_mode: enforcing
sebooleans_enable: []
sebooleans_disable: []
```

Cleanup unused folders in the role:

```bash
rm -rf roles/demo_role_fqcn/{tests,vars,handlers,files,templates}
```

Modify the previous `playbook.yml` file to use the new role:

```yaml
---
- hosts: localhost
  become: true
  vars:
    sebooleans_enable:
      - httpd_can_network_connect
      - httpd_mod_auth_pam
    sebooleans_disable:
      - httpd_enable_cgi
  roles:
    - selinux_manage_FQCN
```

Run the playbook again and check the results:

```bash
$ ansible-playbook playbook.yml
```
This concludes the guided exercise.

# Takeaways

- Collections can be called from roles using the `collections` list defined in `meta/main.yml`.

- Collections can be called from roles using their FQCN directly from the role task

----
**Navigation**
<br>
[Previous Exercise](../3-collections-from-playbook/) - [Next Exercise](../5-collections-from-controller/)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
