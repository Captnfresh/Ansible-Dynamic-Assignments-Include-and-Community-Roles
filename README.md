# Ansible Dynamic Assignments (Include) and Community Roles

## Step-by-Step Guide to Creating Dynamic Configurations
Let’s walk through this step-by-step and make it clear for you. This part is about setting up dynamic assignments using the include module in Ansible and organizing environment-specific configurations

## Step 1: Start a New Branch

1. In your GitHub repository `(ansible-config-mgt)`, create a new branch called `dynamic-assignments`.

```
git checkout -b dynamic-assignments
```
![image](https://github.com/user-attachments/assets/20fdce3e-e1eb-424a-9634-efa907f65624)
This branch will hold our dynamic configuration changes.

2. Create the dynamic-assignments Folder. Create a new folder called `dynamic-assignments` in your repository.

```
mkdir dynamic-assignments
```

3. Inside this folder, create a file named `env-vars.yml`. This file will hold the configurations for including environment variables dynamically.
```
touch dynamic-assignments/env-vars.yml
```
![image](https://github.com/user-attachments/assets/838e4343-2841-4507-b259-363f583bb90b)

By now, your GitHub repository should look like this:
```
├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
```
Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder `env-vars`, then for each environment, create new YAML files which we will use to set variables.

![image](https://github.com/user-attachments/assets/60424e46-4988-47e7-a5ff-e42a65d1de87)

Your layout should now look like this
```
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml
```

4. Now paste the instruction below into the env-vars.yml file.

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```
![image](https://github.com/user-attachments/assets/a442a168-cbf0-4833-8750-ad40d8a079f3)

## Notice 3 things to notice here:

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:

* include_role
* include_tasks
* include_vars


















