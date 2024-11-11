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

5. Since we cannot use both apache and nginx load balancer at the same time, it is advisable to create a condition that enables either one of the two, to do this, Declare a variable in roles/apache/defaults/main.yml file inside the apache role, name the variable enable_apache_lb. Declare another variable that ensures either one of the load balancer is required and set it to false.

```
enable_apache_lb: false
load_balancer_is_required : false
```

![image](https://github.com/user-attachments/assets/5807bbdb-323c-4f4f-a2fb-532ef946f8e7)

![image](https://github.com/user-attachments/assets/f7f34727-a99d-4147-8e17-43cd40c9425e)

6. Create a new playbook in `static-assignments` and call it `loadbalancers.yml`, update it with code below:

![image](https://github.com/user-attachments/assets/42f117bd-35b8-42b0-83af-c124ec610538)

```
---
- hosts: lb
  become: yes
  roles:
    - role: nginx
      when: enable_nginx_lb | bool and load_balancer_is_required | bool
    - role: apache
      when: enable_apache_lb | bool and load_balancer_is_required | bool
```


![image](https://github.com/user-attachments/assets/b7f4d2f5-8043-49d1-bc9d-7bfa643bf6df)

7. Now, inside your general playbook `site.yml` file, dynamically import the load balancer playbook so it can use the roles weve created:
```
---
- hosts: all
  name: Include dynamic variables
  become: yes
  tasks:
    - include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always

- import_playbook: ../static-assignments/common.yml
- import_playbook: ../static-assignments/uat_webservers.yml
- import_playbook: ../static-assignments/loadbalancers.yml
- import_playbook: ../static-assignments/db-servers.yml
```

![image](https://github.com/user-attachments/assets/8d8ff43c-c5ef-4838-88ef-537d7a2ef0ac)

8. To activate load balancer, and enable either of Apache or Nginx load balancer, we can achieve this by setting these in the respective environment's `env-vars` file. Open the `env-vars/uat.yml` file and set it:

```
---
load_balancer_is_required: true
enable_nginx_lb: true
enable_apache_lb: false
```
![image](https://github.com/user-attachments/assets/bccb8d26-693c-4bd5-9408-aa1d73e170f1)
>To use apache, we can set the enable_apache_lb variable to true, and enable_nginx_lb to false. do the same thing for nginx if you want to enable nginx load balancer.


## Step 4: Configuring the apache and Nginx roles to work as load balancer.

## For Apache
In the roles/apache/tasks/main.yml file, we need to include a task that tells ansible to first check if nginx is currently running and enabled, if it is, ansible should first stop and disable nginx before proceeding to install and enable apache. this is to avoid confliction and should always free up the port 80 for the required load balancer. use the code beow to achieve this:

```
- name: Check if nginx is running
  ansible.builtin.service_facts:

- name: Stop and disable nginx if it is running
  ansible.builtin.service:
    name: nginx
    state: stopped
    enabled: no
  when: "'nginx' in services and services['nginx'].state == 'running'"
  become: yes
```
![image](https://github.com/user-attachments/assets/da3bd137-d01f-473c-864e-e9a2dda93321)

### To use apache as a load balancer, we will need to allow certain apache modules that will enable the load balancer. this is the APACHE A2ENMOD

- In the `roles/apache/tasks/configure-debian.yml` file, Create a task to install and enable the required apache a2enmod modules, use the code below :
```
- name: Enable Apache modules
  ansible.builtin.shell:
    cmd: "a2enmod {{ item }}"
  loop:
    - rewrite
    - proxy
    - proxy_balancer
    - proxy_http
    - headers
    - lbmethod_bytraffic
    - lbmethod_byrequests
  notify: restart apache
  become: yes
```
- Create another task to update the apache configurations with required code block needed for the load balancer to function. use the code below :
```
- name: Insert load balancer configuration into Apache virtual host
  ansible.builtin.blockinfile:
  path: /etc/apache2/sites-available/000-default.conf
  block: |
    <Proxy "balancer://mycluster">
      BalancerMember http://<webserver1-ip-address>:80
      BalancerMember http://<webserver2-ip-address>:80
      ProxySet lbmethod=byrequests
    </Proxy>
    ProxyPass "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"
  marker: "# {mark} ANSIBLE MANAGED BLOCK"
  insertbefore: "</VirtualHost>"
notify: restart apache
become: yes
```

![image](https://github.com/user-attachments/assets/c1ad4bda-dddf-496e-8b55-36ba23deae43)

- Enable Apache (in env-vars/uat.yml)

![image](https://github.com/user-attachments/assets/8146c8e2-c3f4-4f47-8d97-461e804ccb54)
 
- Save and create a pull request to merge with the main branch of your github repo.

![image](https://github.com/user-attachments/assets/4747ce92-a383-40e7-bcfb-4540c0db39d1)

![image](https://github.com/user-attachments/assets/b360149e-4e62-4816-8a83-ee593c54f726)

## For Nginx
* In the `roles/nginx/tasks/main.yml` file, create a similar task like we did above to check if apache is active and enabled, if it is, it should disable and stop apache before proceeding with the tasks of installing nginx. use the code below:

```
- name: Check if Apache is running
  ansible.builtin.service_facts:
                     
- name: Stop and disable Apache if it is running
  ansible.builtin.service:
    name: apache2 
    state: stopped
    enabled: no
  when: "'apache2' in services and services['apache2'].state == 'running'"
  become: yes
```

![image](https://github.com/user-attachments/assets/b6307e7e-7866-4d45-b5cf-8710470c7e59)

* In the roles/nginx/handlers/main.yml file, set nginx to always perform the tasks with sudo privileges, use the function : become: yes to achieve this.
  Do the same for all tasks that require sudo privileges

![image](https://github.com/user-attachments/assets/98b571da-2aa0-4230-8ad7-017854d384e7)

* In the `role/nginx/defaults/main.yml` file, uncomment the `nginx_vhosts`, and `nginx_upstream` section.
Under the nginx_vhosts section, ensure you have the same code:
```
nginx_vhosts:
 - listen: "80" # default: "80"
    server_name: "example.com" 
    server_name_redirect: "example.com"
    root: "/var/www/html" 
    index: "index.php index.html index.htm" # default: "index.html index.htm"

# filename: "nginx.conf" # Can be used to set the vhost filename.
                   
    locations:
      - path: "/"
        proxy_pass: "http://myapp1"
                   
# Properties that are only added if defined:
    server_name_redirect: "www.example.com" # default: N/A
    error_page: ""
    access_log: ""
    error_log: ""
    extra_parameters: "" # Can be used to add extra config blocks (multiline).
    template: "{{ nginx_vhost_template }}" # Can be used to override the `nginx_vhost_template` per host.
    state: "present" # To remove the vhost configuration.

nginx_upstreams: 
- name: myapp1
  strategy: "ip_hash" # "least_conn", etc.
  keepalive: 16 # optional
  servers:
    - "172.31.11.6 weight=5"
    - "172.31.11.238 weight=5"
```

![image](https://github.com/user-attachments/assets/cbe22fd7-6664-4758-8591-16e5d284618c)

* Finally, update the `inventory/uat.yml` to include the neccesary details for ansible to connect to each of these servers to perform all the roles we have specified. use the code below:
  Update `roles/nginx/templates/nginx.conf.j2` Comment the line `include {{ nginx_vhost_path }}/*`;


## Step 5 : Configure your webserver roles to install php and all its dependencies , as well as cloning tooling website from github repo.

* In the `roles/webserver/tasks/main.yml` , write the following tasks. use the code below :

```
- name: Install Apache
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
      name: "httpd"
      state: present
                   
- name: Install Git
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
     name: "git"
     state: present
                   
- name: Install EPEL release
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
     cmd: sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y
                   
- name: Install dnf-utils and Remi repository
   remote_user: ec2-user
   become: true
   become_user: root
   ansible.builtin.command:
     cmd: sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
                   
- name: Reset PHP module
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
     cmd: sudo dnf module reset php -y
                   
- name: Enable PHP 7.4 module
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
     cmd: sudo dnf module enable php:remi-7.4 -y
                   
- name: Install PHP and extensions
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
   name:
     - php
     - php-opcache
     - php-gd
     - php-curl
     - php-mysqlnd
   state: present
                   
- name: Install MySQL client
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.yum:
   name: "mysql"
   state: present
                   
- name: Start PHP-FPM service
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
    name: php-fpm
    state: started
                   
- name: Enable PHP-FPM service
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
   name: php-fpm
   enabled: true
                   
- name: Set SELinux boolean for httpd_execmem
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.command:
   cmd: sudo setsebool -P httpd_execmem 1
                   
- name: Clone a repo
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.git:
   repo: https://github.com/citadelict/tooling.git
   dest: /var/www/html
   force: yes
                   
- name: Copy HTML content to one level up
  remote_user: ec2-user
  become: true
  become_user: root
   command: cp -r /var/www/html/html/ /var/www/
                   
- name: Start httpd service, if not started
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.service:
   name: httpd
   state: started
                   
- name: Recursively remove /var/www/html/html directory
  remote_user: ec2-user
  become: true
  become_user: root
  ansible.builtin.file:
   path: /var/www/html/html
   state: absent
```

![image](https://github.com/user-attachments/assets/fce3719f-3248-4df6-a7d8-4aa0c34b1021)

* The code above tells ansible to install apache on the webservers , install git, install php and all its dependencies, clone the website from out github repo,as well ascopy the website files into the /var/www/html directory.
* Create a pull request and merge with your main branch of your github repo.
* Login to your ansible server via terminal and change directory into your ansible project, pull the recent changes done into your server














