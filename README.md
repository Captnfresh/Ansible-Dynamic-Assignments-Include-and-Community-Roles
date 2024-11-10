# Ansible Dynamic Assignments (Include) and Community Roles

## Step-by-Step Guide to Creating Dynamic Configurations
Let’s walk through this step-by-step and make it clear for you. This part is about setting up dynamic assignments using the include module in Ansible and organizing environment-specific configurations

## Step 1: Start a New Branch

1. In your GitHub repository `(ansible-config-mgt)`, create a new branch called `dynamic-assignments`.

```
git checkout -b dynamic-assignments
```
This branch will hold our dynamic configuration changes.

2. Create the dynamic-assignments Folder. Create a new folder called `dynamic-assignments` in your repository.

```
mkdir dynamic-assignments
```

3. Inside this folder, create a file named `env-vars.yml`. This file will hold the configurations for including environment variables dynamically.
```
touch dynamic-assignments/env-vars.yml
```

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
![image](https://github.com/user-attachments/assets/82c6abb6-94e8-4c08-9152-915e9ac7c2c1)


![image](https://github.com/user-attachments/assets/71babc77-7b96-4479-b855-27f500ffc16f)


## Notice 3 things to notice here:

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:

* [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)
* [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)
* [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)

In the same version, variants of import were also introduces, such as:
* [import_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)
* [import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)

2. We made use of a [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop._ with_first_found _ implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.


## Step 2: Update site.yml with dynamic assignments 
1. Update `site.yml` file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

`site.yml` should now look like this.

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```
![image](https://github.com/user-attachments/assets/889d034d-3000-404b-a56d-cd5898e7628b)

### Community Roles
Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

### Download MySQL Ansible Role
You can browse available community roles [here](https://galaxy.ansible.com/ui/). We will be using a [MySQL role developed by geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql)
![image](https://github.com/user-attachments/assets/4fef3456-aeb7-45f8-8d86-2c15da5305fd)


>### Hint: To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ansible-config-mgt directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it – we will be using Jenkins later for a better purpose.

2. On Jenkins-Ansible server make sure that git is installed by running:

```
git --version
cd ansible-config-mgt
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```

![image](https://github.com/user-attachments/assets/e1284b80-0a6f-44c9-bdfc-442830400b85)

3. Inside the `mysql/vars/main.yml`, configure your db credentials. These credentials will be used to connect to our website later on. 

![image](https://github.com/user-attachments/assets/97eb0c33-67b0-472a-a10f-2d2b79252c29)

![image](https://github.com/user-attachments/assets/0319330f-2d68-4156-808a-0925cda6d542)

4. Kindly navigate to `mysql/vars/main.yml`.

```
mysql_root_password: ""
mysql_databases:
  - name: tooling
    encoding: utf8
    collation: utf8_general_ci
mysql_users:
  - name: webaccess
    host: "172.31.0.0/20"
    password: Admin123
    priv: "tooling.*:ALL"
```

![image](https://github.com/user-attachments/assets/1e94d42f-9c26-4886-8fef-75806bbbf52b)

5. Create a new playbook inside static-assignments folder and call it db-servers.yml , update it with the created roles. use the code below.

```
- hosts: db_servers
  become: yes
  vars_files:
    - vars/main.yml
  roles:
    - { role: mysql }
```

![image](https://github.com/user-attachments/assets/8813e1b3-529a-4fbb-b17e-0c39f0b14a4a)

![image](https://github.com/user-attachments/assets/e0a70da8-175e-4a6d-bef3-26e79f8d827f)

6. Now return to playbook which is the `playbooks/site.yml` and reference the newly created `db-servers` playbook, add the code below to import it into the main playbook.

```
import_playbook: ../static-assignments/db-servers.yml
```

![image](https://github.com/user-attachments/assets/589ceaaf-d509-4e5f-8d65-8a51e8c9673f)

7. Now it is time to upload the changes into your GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
![image](https://github.com/user-attachments/assets/7c0e6ed8-d008-47a1-8ccd-59834959fcd8)

![image](https://github.com/user-attachments/assets/36b61810-62eb-434b-8c76-0ab0764d2733)

8. Merge the Code.

![image](https://github.com/user-attachments/assets/054d3b76-e3ed-4b0f-8514-9148a6102a01)


## Step 3: Load Balancer Roles
For this project , we will be making use of NGINX and APACHE as load balancers, so we need to create roles for them using same method as we did for mysql

1. Download and install roles for apache, we can get this role from same source as mysql.

```
ansible-galaxy role install geerlingguy.apache
```
2. Download and install roles for nginx, we can get this role from same source as mysql.

```
ansible-galaxy role install geerlingguy.nginx
```
3. Rename the folder to apache

```
mv geerlingguy.apache/ apache
```
4. Rename the folder to nginx
```
mv geerlingguy.nginx/ nginx
```

![image](https://github.com/user-attachments/assets/e72394a1-5e14-40ad-b1f1-65d2c4efe7a4)














