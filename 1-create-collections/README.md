# Exercise 1 - Installing and Creating Collections

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
    - [Step 1: Installing collections from the command line](#step-1-installing-collections-from-the-command-line)
        - [Preparing the exercise environment](#preparing-the-exercise-environment)
        - [Installing in the default collections path](#installing-in-the-default-collections-path)
        - [Instaling in a custom collections path](#instaling-in-a-custom-collections-path)
        - [Inspecting the contents of the collection](#inspecting-the-contents-of-the-collection)
    - [Step 2: Creating collections from the command line](#step-2-creating-collections-from-the-command-line)
        - [Initializing the Git repository](#initializing-the-git-repository)
    - [Step 3: Adding custom modules and plugins to the collection](#step-3-adding-custom-modules-and-plugins-to-the-collection)
    - [Step 4: Adding custom roles to the collection](#step-4-adding-custom-roles-to-the-collection)
    - [Step 5: Building and installing collections](#step-5-building-and-installing-collections)
    - [Step 6: Testing collections locally](#step-6-testing-collections-locally)
        - [Running the test playbook](#running-the-test-playbook)
        - [Running the local container](#running-the-local-container)
- [Takeaways](#takeaways)

# Objective

This exercise will help users understand how collections are installed, created and customized.
Covered topics:

- Demonstrate the steps involved in the installation of an Ansible Collection from the
command line using the `ansible-galaxy` utility.
- Demonstrate the steps involved in the creation of a new collection with the `ansible-galaxy`
utility.
- Demonstrate the creation of a custom role inside the newly created collection.
- Demonstrate the creation of a new custom plugin (basic Ansible module) in the newly created collection.

# Guide

## Step 1: Installing collections from the command line

Ansible Collections can be searched and installed from Ansible Galaxy and Red Hat Automation Hub.
Once installed, a collection can be used locally and its plugins, modules, and roles can be imported
and executed in complex Ansible-based projects.

### Preparing the exercise environment

Create a directory in your lab named `workspace\ansible-collections` and cd into it.

```bash
mkdir -p ~/workspace/ansible_collections
cd ~/workspace/ansible_collections
```

Collection have two default lookup paths that are searched:

- User scoped path `/home/<username>/.ansible/collections`

- System scoped path `/usr/share/ansible/collections`

> **TIP**: Users can customized the collections path by modifying the `collections_path` key in the
> `ansible.cfg` file or by setting the environment variable `ANSIBLE_COLLECTIONS_PATHS` with the desired
> search path.

### Installing in the default collections path

First, we demonstrate how to install a collection in the user scoped path.For the sake of simplicity we are going to use the collection [newswangerd.collection_demo](https://galaxy.ansible.com/newswangerd/collection_demo), a basic collection created for demo purposes.

It contains basic roles and a very simple module and is a good example to understand how a collection works without getting involved in modules or roles logic.

Install the collection using the command `ansible-galaxy collection install` with no extra options:

```bash
$ ansible-galaxy collection install newswangerd.collection_demo
Process install dependency map
Starting collection install process
Installing 'newswangerd.collection_demo:1.0.10' to '/home/<username>/.ansible/collections/ansible_collections/newswangerd/collection_demo'
```

The collection is now installed in the user home directory and can be used in playbooks and roles.

### Instaling in a custom collections path

Install the collection in the current working directory using the `-p` flag followed by the custom installation path.

```bash
ansible-galaxy collection install -p . newswangerd.collection_demo
```

> **NOTE**: When installing on custom paths not included in the collections search path a standard warning message is issued:
>
> ```bash
>  [WARNING]: The specified collections path '/home/gbsalinetti/Labs/collections-lab' is not part of the configured Ansible collections paths
> '/home/gbsalinetti/.ansible/collections:/usr/share/ansible/collections'. The installed collection won't be picked up in an Ansible run.
> ```

The installed path follows the standard pattern `ansible_collections/<author>/<collection>`.

Run the `tree` command to inspect the contents:

```bash
$ tree
.
└── ansible_collections
    └── newswangerd
        └── collection_demo
            ├── docs
            │   └── test_guide.md
            ├── FILES.json
            ├── MANIFEST.json
            ├── plugins
            │   └── modules
            │       └── real_facts.py
            ├── README.md
            ├── releases
            │   ├── newswangerd-collection_demo-1.0.0.tar.gz
            │   ├── newswangerd-collection_demo-1.0.1.tar.gz
            │   ├── newswangerd-collection_demo-1.0.2.tar.gz
            │   ├── newswangerd-collection_demo-1.0.3.tar.gz
            │   ├── newswangerd-collection_demo-1.0.4.tar.gz
            │   └── newswangerd-collection_demo-1.0.5.tar.gz
            └── roles
                ├── deltoid
                │   ├── meta
                │   │   └── main.yaml
                │   ├── README.md
                │   └── tasks
                │       └── main.yml
                └── factoid
                    ├── meta
                    │   └── main.yaml
                    ├── README.md
                    └── tasks
                        └── main.yml
```

### Inspecting the contents of the collection

Collections have a standard structure that can can hold modules, plugins, roles and playbooks.

```bash
collection/
├── docs/
├── galaxy.yml
├── plugins/
│   ├── modules/
│   │   └── module1.py
│   ├── inventory/
│   └── .../
├── README.md
├── roles/
│   ├── role1/
│   ├── role2/
│   └── .../
├── playbooks/
│   ├── files/
│   ├── vars/
│   ├── templates/
│   └── tasks/
└── tests/
```

A short description of the collection structure:

- The `plugins` folder holds plugins, modules, and module_utils that can be reused in playbooks and roles.
- The `roles` folder hosts custom roles, while all collection playbooks must be stored in the `playbooks` folder.
- The `docs` folder can be used for the collections documentation, as well as the main `README.md` file that is
  used to describe the collection and its content.
- The `tests` folder holds tests written for the collection.
- The `galaxy.yml` file is a YAML text file that contains all the metadata used in the Ansible Galaxy hub to index the collection.
  It is also used to list collection dependencies, if there are any.

When a collection is downloaded with the `ansible-galaxy collection install` two more files are installed:

- `MANIFEST.json`, holding additional Galaxy metadata in JSON format.

- `FILES.json`, a JSON object containing all the files SHA256 checksum.

## Step 2: Creating collections from the command line

Users can create their own collections and populate them with roles, playbook, plugins and modules.
User defined collections skeleton can be created manually or it can be authored with the
`ansible-galaxy collection init` command. This will create a standard skeleton that can be
customized lately.

Create the following collection:

```bash
cd ~/workspace/ansible_collections
ansible-galaxy collection init workshop.demo_collection
```
The collection name always follows the pattern `<namespace.collection>`. The above example creates
the `demo_collection` in the `workshop` namespace.

The command created the following skeleton:

```bash
$ tree ~/workspace/ansible_collections/workshop/demo_collection/
ansible_collections/workshop/demo_collection/
├── docs
├── galaxy.yml
├── plugins
│   └── README.md
├── README.md
└── roles
```

The skeleton is really minimal. Besides the template README files, a template `galaxy.yml` file is created to define Galaxy metadata.

## Step 3: Adding custom modules and plugins to the collection

Collections can be customized with different kinds of plugins and modules. For a complete list please refer to the `README.md` file in the `plugins` folder.

In this workshop we are going to have 2 modules, we will create one and use the timezone module that already exists in the community collection. these will be installed into the `plugins/modules` directory.

You can refer the README.md file to know the list of the directories that can be created under plugins directory.

As we will be copying a module file, we first need to create a directory with name modules under plugins directory:

```bash
cd ~/workspace/ansible_collections/workshop/demo_collection/plugins
mkdir modules && cd modules
```

Create the `demo_hello.py` module in the new folder. The module code is available in the `solutions/modules`
folder of this exercise.

```bash
cp ~/exercises/ansible_Automation-Hub/1-create-collections/solutions/modules/demo_hello.py .
```

The `demo_hello` module says Hello in different languages to custom defined users. Take your time to look at the module code and understand its behavior.

```bash
#!/usr/bin/python


ANSIBLE_METADATA = {
    'metadata_version': '1.0',
    'status': ['preview'],
    'supported_by': 'community'
}

DOCUMENTATION = '''
---
module: demo_hello
short_description: A module that says hello in many languages
version_added: "2.8"
description:
  - "A module that says hello in many languages."
options:
    name:
        description:
          - Name of the person to salute. If no value is provided the default
            value will be used.
        required: false
        type: str
        default: John Doe
author:
    - Gianni Salinetti (@giannisalinetti)
'''

EXAMPLES = '''
# Pass in a custom name
- name: Say hello to Linus Torwalds
  demo_hello:
    name: "Linus Torwalds"
'''

RETURN = '''
fact:
  description: Hello string
  type: str
  sample: Hello John Doe!
'''

import random
from ansible.module_utils.basic import AnsibleModule


FACTS = [
    "Hello {name}!",
    "Bonjour {name}!",
    "Hola {name}!",
    "Ciao {name}!",
    "Hallo {name}!",
    "Hei {name}!",
]


def run_module():
    module_args = dict(
        name=dict(type='str', default='John Doe'),
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    result = dict(
        changed=False,
        fact=''
    )

    result['fact'] = random.choice(FACTS).format(
        name=module.params['name']
    )

    if module.check_mode:
        return result

    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```

An Ansible module is basically an implementation of the AnsibleModule class created and executed in a minimal function called `run_module()`. As you can see, a module has a `main()` function, like a plain Python executable. Anyway, it is
not meant to be executed independently.

Next download the Ansible timezone module python file:

```bash
curl -O https://raw.githubusercontent.com/ansible/ansible/stable-2.9/lib/ansible/modules/system/timezone.py
```

```bash
ls -l
-rw-rw-r--. 1 student student  1355 Jan 18 09:04 demo_hello.py
-rw-rw-r--. 1 student student 36525 Jan 18 09:05 timezone.py
```

Likewise you can place your own modules under plugins/modules directory and other plugins lookups, filters, and so on in their respecting directory by creating them.

## Step 4: Adding custom roles to the collection

The last step of this exercise will be focused on a role creation inside the custom collection. We will deploy a basic role that uses the previous modules and a regular task.

> **TIP**: If you want to speed up the lab you can copy the completed role from the exercise `solutions/roles` folder.

Generate the new role skeleton using the `ansible-galaxy init` command:

```bash
cd ~/workspace/ansible_collections/workshop/demo_collection
ansible-galaxy init --init-path roles custom_role
```

Create the following tasks in the `roles/custom_role/tasks/main.yml` file:

```yaml
---
# tasks file for demo_custom_role
- name: Ensure timezone package is present in the host
  ansible.builtin.dnf:
    name: tzdata
    state: present
  become: true

- name: Generate greeting and store result
  workshop.demo_collection.timezone:
    name: Asia/Tokyo

- name: Generate greeting and store result
  workshop.demo_collection.demo_hello:
    name: ""
  register: demo_greeting

- name: Print output from demo_hello module task
  ansible.builtin.debug:
    var: demo_greeting
```

Notice the usage of the `demo_hello` module, installed in the collection, to generate the greeting string.

> **NOTE**: When a collection role calls a module in the same collection namespace, the module is automatically resolved.

Create the following variables in the `roles/custom_role/defaults/main.yml`:

```yaml
---
# defaults file for custom_role
friend_name: "John Doe"
timezone: "Asia/Tokyo"
```
The skeleton generates a complete structure files and folder. We can clean up the unused ones:

```bash
rm -rf roles/custom_role/{handlers,vars,tests}
```

Customize the `roles/custom_role/meta/main.yml` file to define Galaxy metadata and potential dependencies of the role. Use this sample minimal content:
```yaml
galaxy_info:
  author: Ansible Automation Platform Workshop
  description: Custom role demo
  company: Red Hat

  license: Apache-2.0

  min_ansible_version: 2.8


  platforms:
    - name: Fedora
      versions:
      - 31
      - 32
      - 33

  galaxy_tags: ["demo", "workshop"]

dependencies: []
```
Customize the `meta/runtime.yml` file to define the minimum version of ansible needed. Use this sample minimal content:
```yaml
---
# Collections must specify a minimum required ansible version to upload
# to galaxy
requires_ansible: '>=2.9.10'
```

## Step 5: Building and installing collections

Once completed the creation task we can build the collection and generate a .tar.gz file that can be installed locally or uploaded to Galaxy.

From the collection folder run the following command:

```bash
cd ~/workspace/ansible_collections/workshop/demo_collection
ansible-galaxy collection build
```

The above command will create the file `workshop-demo_collection-1.0.0.tar.gz`. Notice the semantic x.y.z versioning.

Once created the file can be installed in the `COLLECTIONS_PATH` to be tested locally:

```bash
ansible-galaxy collection install workshop-demo_collection-1.0.0.tar.gz
```

By default the collection will be installed in the `~/.ansible/collections/ansible_collections` folder. Now the collection can be tested locally.

## Step 6: Testing collections locally

Create the `collections_test` folder to execute the local test:

```bash
mkdir ~/workspace/ansible_collections/workshop/collections_test
cd ~/workspace/ansible_collections/workshop/collections_test
```

Create a basic `playbook.yml` file with the following contents:

```bash
cat > playbook.yml << EOF
---
- hosts: localhost
  become: true
  vars:
    timezone: Asia/Tokyo
    friend_name: Heisenberg
  tasks:
    - import_role:
        name: workshop.demo_collection.custom_role
EOF
```

### Running the test playbook

Run the test playbook.

```bash
$ ansible-playbook playbook.yml
```

# Takeaways

- Collections can be installed from Galaxy of from Red Hat Automation Hub. Default collections search paths or custom paths can be used.

- Collections can be created using the `ansible-galaxy collection init` command. Users can develop collections contents accordingly to their needs and business logic.

- Collections plugins can be either any kind of Ansible plugins or modules. Modules are often developed inside collection to create an autonomous lifecycle from the main Ansible upstream.

- Collection roles can use local collections plugins and modules.

----
**Navigation**
<br>
[Next Exercise](../2-collections-upload-to-Automation-Hub)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
