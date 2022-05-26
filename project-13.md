
### PROJECT 13: Ansible Dynamic Assignments (Include) and Community Roles


#### Introducing Dynamic Assignment Into the structure

In the ```https://github.com/<your-name>/ansible-config-mgt GitHub repository ```, start a new branch and call it dynamic-assignments.
  
![Screen Shot 2021-06-14 at 9 58 45 AM](https://user-images.githubusercontent.com/44268796/121904430-2bdfae80-ccf7-11eb-9a99-2de7ff17268a.png)


Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. site.yml will be instructed to include this playbook later. 

GitHub will have the following structure at this point:
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

Create a folder to keep each environment’s variables file and name it as env-vars. Then for each environment, create new YAML files which will be used to set variables.

The layout should now look like this:
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

![Screen Shot 2021-06-14 at 10 11 26 AM](https://user-images.githubusercontent.com/44268796/121906214-e3c18b80-ccf8-11eb-9c3a-1873f196b1ef.png)


Paste the instruction below into the env-vars.yml file:
```yml
---
- name: collate variables from env specific file, if it exists
  include_vars: "{{ item }}"
  with_first_found:
    - "{{ playbook_dir }}/../env_vars/{{ "{{ inventory_file }}.yml"
    - "{{ playbook_dir }}/../env_vars/default.yml"
  tags:
    - always
```


#### Update site.yml with dynamic assignments

Update site.yml file to make use of the dynamic assignment. (At this point, testing cannot be done yet. This is just setting the stage for what is to come).

site.yml should now look like this:
```yml
---
- name: Include dynamic variables 
  hosts: all
  tasks:
    - import_playbook: ../static-assignments/common.yml 
    - include_playbook: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- name: Webserver assignment
  hosts: webservers
    - import_playbook: ../static-assignments/webservers.yml
```

![Screen Shot 2021-06-14 at 11 19 27 AM](https://user-images.githubusercontent.com/44268796/121916714-669b1400-cd02-11eb-9157-b0447d9e95f8.png)

#### Community Roles

It is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users.

#### Download Mysql Ansible Role

Browse the available community roles [here](https://galaxy.ansible.com/home)

This project will make use of a MySQL role developed by [geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql).

On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run:
```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```


![Screen Shot 2021-06-14 at 11 37 50 AM](https://user-images.githubusercontent.com/44268796/121919367-f5109500-cd04-11eb-9e3d-6c260876bd6c.png)

Inside roles directory, create a new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql:
```
ansible-galaxy install geerlingguy.mysql
mv geerlingguy.mysql/ mysql
```

![Screen Shot 2021-06-14 at 11 52 53 AM](https://user-images.githubusercontent.com/44268796/121921500-0f4b7280-cd07-11eb-9efb-b1ff8085288c.png)

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website. The main.yml file in the defaults folder of roles directory is edited as follows:


![Screen Shot 2021-06-14 at 12 03 55 PM](https://user-images.githubusercontent.com/44268796/121923280-c3013200-cd08-11eb-96b6-df413f3805ff.png)



Now it is time to upload the changes into GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```

Create a Pull Request and merge it to main branch on GitHub.

#### Load Balancer roles

In order to be able to choose which Load Balancer to use, Nginx or Apache, we will need to have two roles:

1. Nginx
2. Apache

- With the experience gained from the projects on Ansible so far, it is understood that roles could either be developed or recreated easily from available roles in the community. 

![Screen Shot 2021-06-14 at 2 12 32 PM](https://user-images.githubusercontent.com/44268796/121939067-91916200-cd1a-11eb-9f86-f4aac4378082.png)



- After creating the roles, update both static-assignment and site.yml files to refer the roles.

Important Hints:

- Since both Nginx and Apache load balancer cannot be used together, add a condition to enable either one - this is where the use of variables is handy.
- Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.
- Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.
- Declare another variable in both roles load_balancer_is_required and set its value to false as well
- Update both assignment and site.yml files respectively

On main.yml in Nginx:

![Screen Shot 2021-06-14 at 2 23 16 PM](https://user-images.githubusercontent.com/44268796/121940409-13ce5600-cd1c-11eb-82db-173fc1fb3509.png)


On main.yml in Apache:

![Screen Shot 2021-06-14 at 2 25 48 PM](https://user-images.githubusercontent.com/44268796/121940833-850e0900-cd1c-11eb-9f45-c8d5fd141bb6.png)

Create loadbalancers.yml file in static-assignments folder and copy the following code:
```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

Update site.yml file with the following code:
```
- name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
```

![Screen Shot 2021-06-14 at 2 32 24 PM](https://user-images.githubusercontent.com/44268796/121941519-63615180-cd1d-11eb-910c-a4d68d487686.png)


Now the env-vars\uat.yml file can be used to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

- Activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

![Screen Shot 2021-06-14 at 3 38 45 PM](https://user-images.githubusercontent.com/44268796/121949580-9e1bb780-cd26-11eb-92ae-6d568ed8c85e.png)


- The same must work with apache LB, so it can be switched by setting respective environmental variable to true and other to false.

![Screen Shot 2021-06-14 at 3 43 19 PM](https://user-images.githubusercontent.com/44268796/121950165-47fb4400-cd27-11eb-86a9-72e60f3cc742.png)

- To test this, update inventory for each environment and run Ansible against each environment.

Blockers: The playbook was not running and there were errors in the env-vars.yml file. With the help of the mentorship team at Darey.io, I incorporated some changes to the env-vars.yml file. I updated the file with the following code:

```yml
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - prod.yml
            - stage.yml
            - default.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
  ```
  
  In addition, the code on site.yml was edited to:
  
  ```yml
  ---
- hosts: all
- name:  dynamic variables 
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

- hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/uat-webservers.yml
```

  Another change was to rename the files under the inventory folder by removing the .yml extension:
  
  ![Screen Shot 2021-06-18 at 11 52 00 AM](https://user-images.githubusercontent.com/44268796/122587249-99f3e080-d02b-11eb-92f8-cdd52d0362d4.png)

  After the above changes, the playbook executed successfully across all the servers. 
  
  ![Screen Shot 2021-06-18 at 11 55 46 AM](https://user-images.githubusercontent.com/44268796/122587864-59489700-d02c-11eb-90b0-c63bf30447e3.png)






















