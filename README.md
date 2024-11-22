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

```---
# Play for including dynamic variables
- name: Include dynamic variables
  hosts: all
  become: yes
  tasks:
    - name: Load dynamic variables
      include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always
- import_playbook: ../static-assignments/uat-webservers.yml
- import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancer_is_required
- import_playbook: ../static-assignments/db-servers.yml
```
![image](https://github.com/user-attachments/assets/43aca586-e430-44a8-999f-7ca4bcf377bc)

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
---
- hosts: db
  become: yes
  vars_files:
    - vars/main.yml
  roles:
    - role: mysql
```

![image](https://github.com/user-attachments/assets/8813e1b3-529a-4fbb-b17e-0c39f0b14a4a)

![image](https://github.com/user-attachments/assets/34010d26-be9e-4108-9bc8-0e2b5b7274e1)

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

5. Since we cannot use both apache and nginx load balancer at the same time, it is advisable to create a condition that enables either one of the two, to do this, Declare a variable in `roles/apache/defaults/main.yml` file inside the apache role, name the variable `enable_apache_lb`. Declare another variable that ensures either one of the load balancer is required and set it to false.

```
enable_apache_lb: false
load_balancer_is_required : false
```

![image](https://github.com/user-attachments/assets/ce6f7279-d2ee-4252-b4d0-74e527b1ac7f)

6. Create a new playbook in `static-assignments` and call it `loadbalancers.yml`, update it with code below:

![image](https://github.com/user-attachments/assets/42f117bd-35b8-42b0-83af-c124ec610538)

```
---
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```


![image](https://github.com/user-attachments/assets/96d3f95d-f5f7-4ec6-b507-80186c23cc07)

7. Now, inside your general playbook `site.yml` file, dynamically import the load balancer playbook so it can use the roles weve created:
```
---
# Play for including dynamic variables
- name: Include dynamic variables
  hosts: all
  become: yes
  tasks:
    - name: Load dynamic variables
      include_tasks: ../dynamic-assignments/env-vars.yml
      tags:
        - always
- import_playbook: ../static-assignments/uat-webservers.yml
- import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancer_is_required
- import_playbook: ../static-assignments/db-servers.yml
```

![image](https://github.com/user-attachments/assets/2385b735-7699-4c04-96d9-950792d89654)

8. To activate load balancer, and enable either of Apache or Nginx load balancer, we can achieve this by setting these in the respective environment's `env-vars` file. Open the `env-vars/uat.yml` file and set it:

```
---
enable_nginx_lb: true
enable_apache_lb: false
load_balancer_is_required: true
```
![image](https://github.com/user-attachments/assets/6672ab18-7c24-4fa5-98d8-334816ef81ea)
>To use apache, we can set the enable_apache_lb variable to true, and enable_nginx_lb to false. do the same thing for nginx if you want to enable nginx load balancer.


## Step 4: Configuring the apache and Nginx roles to work as load balancer.

## For Apache
In the roles/apache/tasks/main.yml file, we need to include a task that tells ansible to first check if nginx is currently running and enabled, if it is, ansible should first stop and disable nginx before proceeding to install and enable apache. this is to avoid confliction and should always free up the port 80 for the required load balancer. use the code beow to achieve this:

```
---
# Include variables and define needed variables.
- name: Include OS-specific variables.
  include_vars: "{{ ansible_os_family }}.yml"
  when: enable_nginx

- name: Include variables for Amazon Linux.
  include_vars: "AmazonLinux.yml"
  when:
    - ansible_distribution == "Amazon"
    - ansible_distribution_major_version == "NA"

- name: Define apache_packages.
  set_fact:
    apache_packages: "{{ __apache_packages | list }}"
  when: apache_packages is not defined

# Setup/install tasks.
- include_tasks: "setup-{{ ansible_os_family }}.yml"

# Figure out what version of Apache is installed.
- name: Get installed version of Apache.
  command: "{{ apache_daemon_path }}{{ apache_daemon }} -v"
  changed_when: false
  check_mode: false
  register: _apache_version

- name: Create apache_version variable.
  set_fact:
    apache_version: "{{ _apache_version.stdout.split()[2].split('/')[1] }}"

- name: Include Apache 2.2 variables.
  include_vars: apache-22.yml
  when: "apache_version.split('.')[1] == '2'"

- name: Include Apache 2.4 variables.
  include_vars: apache-24.yml
  when: "apache_version.split('.')[1] == '4'"

# Configure Apache.
- name: Ensure Apache is started for UAT if enabled
  service:
    name: apache2
    state: started
    enabled: true
  when:
    - enable_apache_lb
    - load_balancer_is_required

- name: Stop Apache if not required
  service:
    name: apache2
    state: stopped
  when:
    - not enable_apache_lb
    - load_balancer_is_required

```
![image](https://github.com/user-attachments/assets/c4fb92ae-59f6-4124-bb7c-67e630f77437)

![image](https://github.com/user-attachments/assets/e12c45c4-fc33-4c79-95de-5d725edb1df7)


### To use apache as a load balancer, we will need to allow certain apache modules that will enable the load balancer. this is the APACHE A2ENMOD

- In the `roles/apache/tasks/configure-debian.yml` file, Create a task to install and enable the required apache a2enmod modules, use the code below :
```
---
- name: Configure Apache.
  lineinfile:
    dest: "{{ apache_server_root }}/ports.conf"
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
    state: present
    mode: 0644
  with_items: "{{ apache_ports_configuration_items }}"
  notify: restart apache

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

- name: Insert load balancer configuration into Apache virtual host
  ansible.builtin.blockinfile:
  path: /etc/apache2/sites-available/000-default.conf
  block: |
    <Proxy "balancer://mycluster">
      BalancerMember http://Private IP of UAT webserver 1:80
      BalancerMember http://Private IP of UAT webserver 2:80
      ProxySet lbmethod=byrequests
    </Proxy>
    ProxyPass "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"
  marker: "# {mark} ANSIBLE MANAGED BLOCK"
  insertbefore: "</VirtualHost>"
notify: restart apache
become: yes

- name: Disable Apache mods.
  file:
    path: "{{ apache_server_root }}/mods-enabled/{{ item }}.load"
    state: absent
  with_items: "{{ apache_mods_disabled }}"
  notify: restart apache

- name: Check whether certificates defined in vhosts exist.
  stat: "path={{ item.certificate_file }}"
  register: apache_ssl_certificates
  with_items: "{{ apache_vhosts_ssl }}"
  no_log: "{{ apache_ssl_no_log }}"

- name: Add apache vhosts configuration.
  template:
    src: "{{ apache_vhosts_template }}"
    dest: "{{ apache_conf_path }}/sites-available/{{ apache_vhosts_filename }}"
    owner: root
    group: root
    mode: 0644
  notify: restart apache
  when: apache_create_vhosts | bool

- name: Add vhost symlink in sites-enabled.
  file:
    src: "{{ apache_conf_path }}/sites-available/{{ apache_vhosts_filename }}"
    dest: "{{ apache_conf_path }}/sites-enabled/{{ apache_vhosts_filename }}"
    state: link
    mode: 0644
    force: "{{ ansible_check_mode }}"
  notify: restart apache
  when: apache_create_vhosts | bool

- name: Remove default vhost in sites-enabled.
  file:
    path: "{{ apache_conf_path }}/sites-enabled/{{ apache_default_vhost_filename }}"
    state: absent
  notify: restart apache
  when: apache_remove_default_vhost
```

- Enable Apache (in env-vars/uat.yml)

  ```
  ---
    enable_nginx_lb: false
    enable_apache_lb: true
    load_balancer_is_required: true
  ```

![image](https://github.com/user-attachments/assets/b7c32878-7de8-4079-a136-445292c5bb02)
 
- Save and create a pull request to merge with the main branch of your github repo.

![image](https://github.com/user-attachments/assets/4747ce92-a383-40e7-bcfb-4540c0db39d1)

![image](https://github.com/user-attachments/assets/b360149e-4e62-4816-8a83-ee593c54f726)

## For Nginx
* In the `roles/nginx/tasks/main.yml` file, create a similar task like we did above to check if apache is active and enabled, if it is, it should disable and stop apache before proceeding with the tasks of installing nginx. use the code below:

```
---
- name: Check if Apache is running
  ansible.builtin.service_facts:

- name: Stop and disable Apache if it is running
  ansible.builtin.service:
    name: apache2
    state: stopped
    enabled: no
  when: "'apache2' in services and services['apache2'].state == 'running'"
  become: yes


# Variable setup.
- name: Include OS-specific variables.
  include_vars: "{{ ansible_os_family }}.yml"
  become: yes

- name: Define nginx_user.
  set_fact:
    nginx_user: "{{ __nginx_user }}"
  when: nginx_user is not defined
  become: yes

# Setup/install tasks.
- include_tasks: setup-RedHat.yml
  when: ansible_os_family == 'RedHat' or ansible_os_family == 'Rocky' or ansible_os_family == 'AlmaLinux'


- include_tasks: setup-Ubuntu.yml
  when: ansible_distribution == 'Ubuntu'


- include_tasks: setup-Debian.yml
  when: ansible_os_family == 'Debian'


- include_tasks: setup-FreeBSD.yml
  when: ansible_os_family == 'FreeBSD'


- include_tasks: setup-OpenBSD.yml
  when: ansible_os_family == 'OpenBSD'


- include_tasks: setup-Archlinux.yml
  when: ansible_os_family == 'Archlinux'


- include_tasks: setup-Suse.yml
  when: ansible_os_family == 'Suse'


# Vhost configuration.
- import_tasks: vhosts.yml
  become: yes

# Nginx setup.
- name: Copy nginx configuration in place.
  template:
    src: "{{ nginx_conf_template }}"
    dest: "{{ nginx_conf_file_path }}"
    owner: root
    group: "{{ root_group }}"
    mode: 0644
  notify:
    - reload nginx
  become: yes

- name: Ensure nginx service is running as configured.
  service:
    name: nginx
    state: "{{ nginx_service_state }}"
    enabled: "{{ nginx_service_enabled }}"
  become: yes
```

* In the roles/nginx/handlers/main.yml file, set nginx to always perform the tasks with sudo privileges, use the function : become: yes to achieve this.
  Do the same for all tasks that require sudo privileges

![image](https://github.com/user-attachments/assets/98b571da-2aa0-4230-8ad7-017854d384e7)

* In the `role/nginx/defaults/main.yml` file, uncomment the `nginx_vhosts`, and `nginx_upstream` section.
Under the nginx_vhosts section, ensure you have the same code:
```
nginx_vhosts:
# Example vhost below, showing all available options:
- listen: "80" # default: "80"
  server_name: "example.com" # default: N/A
  root: "/var/www/html" # default: N/A
  index: "index.php index.htm" # default: "index.html index.htm"
  locations:
          - path: "/"
            proxy_pass: "http://myapp1"
    # filename: "example.com.conf" # Can be used to set the vhost filename.
  server_name_redirect: "www.example.com"
  error_page: ""
  access_log: ""
  error_log: ""
  extra_parameters: ""
  template: "{{ nginx_vhost_template }}"
  state: "present"
  become: yes

nginx_upstreams:
- name: myapp1
  strategy: "ip_hash" # "least_conn", etc.
  keepalive: 16 # optional
  servers:
    - "172.31.28.208 weight=5"
    - "172.31.24.16 weight=5"
  become: yes

nginx_log_format: |-
  '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"'
become: yes
```

* Finally, update the `inventory/uat.yml` to include the neccesary details for ansible to connect to each of these servers to perform all the roles we have specified. use the code below. Update `roles/nginx/templates/nginx.conf.j2` Comment the line `include {{ nginx_vhost_path }}/*`;

```

     {% for vhost in nginx_vhosts %}
    server {
        listen {{ vhost.listen }};
        server_name {{ vhost.server_name }};
        root {{ vhost.root }};
        index {{ vhost.index }};

    {% for location in vhost.locations %}
        location {{ location.path }} {
            proxy_pass {{ location.proxy_pass }};
        }
    {% endfor %}
  }
{% endfor %}

```
![image](https://github.com/user-attachments/assets/9f16d50c-9bef-4eeb-a6b3-c0533a5902fb)


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
    repo: https://github.com/Captnfresh/tooling.git
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

* The code above tells ansible to install apache on the webservers , install git, install php and all its dependencies, clone the website from our github repo,as well as copy the website files into the /var/www/html directory.
* Create a pull request and merge with your main branch of your github repo.
* Login to your ansible server via terminal and change directory into your ansible project, pull the recent changes done into your server


## Final Test and Validation

1. Setup Inventory for Each Environment: Update inventory files to include environment-specific servers.

```
[uat_webservers]
<server1-ip-address> ansible_ssh_user=ec2-user
<server2-ip-address> ansible_ssh_user=ec2-user

[lb]
<lb-instance-ip> ansible_ssh_user=ubuntu 

[db]
<db-instance-ip> ansible_ssh_user=ubuntu  
```

2. Run Playbooks: Execute playbooks for different environments to validate that variables and roles are correctly applied.

```
ansible-playbook -i inventory/uat.yml playbooks/site.yml
```
![image](https://github.com/user-attachments/assets/7a023751-af07-4441-a220-37665eef3a7f)

3. Test Loadbalancer public IP on browser:

```
http://<Public IP address of LB server>/index.php
```
![image](https://github.com/user-attachments/assets/7acdcec3-b9d2-49d5-aa0f-2b70500520e5)



# Challenges Faced and Solutions

## 1. Host Pattern Mismatch
* Challenge: While executing my playbook, I encountered the warning:

```
[WARNING]: Could not match supplied host pattern, ignoring: uat_webservers
```
The play was skipped entirely because no hosts matched the group specified in the hosts directive.

* Solution: I traced the issue to an incorrect or missing group definition in my inventory file. To resolve it:
  - Verified and updated the uat_webservers group in the inventory file to include the correct host IPs.
  - Ensured that the inventory file was structured correctly and aligned with the playbook's hosts directive.

Example of the corrected inventory file:
```
[uat_webservers]
172.31.24.16
172.31.28.208
172.31.81.222
172.31.91.204
```

## 2. Reserved Variable Name Conflict
* Challenge: Ansible threw warnings about using become as a variable name, which is reserved for privilege escalation.

* Solution:
  - Renamed the variable to something more descriptive and non-conflicting, such as privilege_escalation.
  - Updated all references in the variable files and playbooks accordingly.
  - This eliminated the warning and maintained clarity in the variable’s purpose.

## 3. Role Dependency Confusion
* Challenge: Imported community roles from Ansible Galaxy, but some roles failed due to unmet dependencies. This led to errors during playbook execution.

* Solution:
  - Reviewed the meta/main.yml file of each role to identify dependencies.
  - Used the following command to install the missing dependencies:
    ```
    ansible-galaxy install -r requirements.yml
    ```  
    
## 4. Syntax Errors in Dynamic Includes
* Challenge: Dynamic includes (include_tasks) failed due to subtle YAML syntax errors, particularly around indentation. These errors caused hours of debugging frustration.

* Solution:
  - Adopted YAML linting tools to validate playbooks and variable files before execution.
    Example command:
    ```
    yamllint playbook.yml
    ```
  - Implemented peer reviews and visual inspection for consistent indentation across all files.


## 5. Community Role Customization
* Challenge: The default parameters in some community roles didn't align with my project requirements, leading to incorrect configurations.

* Solution:
  - Overrode default parameters by passing custom variables directly in the playbook.
  - Created a role-specific variable file in group_vars or host_vars to provide better control and modularity.


## 6. Managing Large Variable Files
* Challenge: Managing environment-specific variables in large YAML files became cumbersome, leading to misconfigurations.

* Solution:
  - Organized variables into separate files based on environments (e.g., uat.yml, prod.yml).
  - Dynamically included the appropriate variable file using:
 
    ```
    - name: Include environment-specific variables
      include_vars: "{{ playbook_dir }}/env-vars/{{ env }}.yml"
    ```
  - Introduced comments and consistent naming conventions to improve readability.



## 7. Debugging and Testing Community Roles
* Challenge: Debugging a failing role was difficult without a clear understanding of the imported role's structure.

* Solution:
  - Reviewed the role's documentation and code structure to identify expected inputs and outputs.
  - Added the -vvv flag to Ansible commands to generate detailed logs for troubleshooting:
 
  ```
  ansible-playbook site.yml -vvv
  ```




# Conclusion

These challenges provided a deep dive into troubleshooting and refining Ansible playbooks. By addressing these issues methodically, I not only completed the project successfully but also gained a stronger understanding of Ansible's dynamic configurations, modularity, and community contributions.





