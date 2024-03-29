# Chapter 3. Managing Variables and Facts
Write playbooks that use variables to simplify management of the playbook and facts to reference information about managed hosts.
# Managing Variables
 - Objectives
     - Create and reference variables that affect particular hosts or host groups, the play, or the global environment, and describe how variable precedence works.
# Introduction to Ansible Variables
Ansible supports variables that can be used to store values that can then be reused throughout files in an Ansible project. This can simplify the creation and maintenance of a project and reduce the number of errors.

Variables provide a convenient way to manage dynamic values for a given environment in your Ansible project. Examples of values that variables might contain include:

- Users to create

- Packages to install

- Services to restart

- Files to remove

- Archives to retrieve from the internet

# Naming Variables
Variable names must start with a letter, and they can only contain letters, numbers, and underscores.

The following table illustrates the difference between invalid and valid variable names.

![Screenshot (524)](https://github.com/Salmamohamedm/Red-Hat-Enterprise-Linux-Automation-with-Ansible-9.0/assets/109488469/8ebd946b-3da8-481d-b41a-ed4619ec0dcf)

# Defining Variables
  Variables can be defined in a variety of places in an Ansible project. If a variable is set using the same name in two places, and those settings have different values, precedence 
  determines which value is used.

You can set a variable that affects a group of hosts or only individual hosts. Some variables are facts that can be set by Ansible based on the configuration of a system. Other variables can be set inside the playbook, and affect one play in that playbook, or only one task in that play.


You can also set extra variables on the ansible-navigator run command line by using the --extra-vars or -e option and specifying those variables, and they override all other values for that variable name.
```
ansible-navigator run -m playbook playbook.yml --extra-vars "variable1=value1 variable2=value2"
```
The following simplified list shows ways to define a variable, ordered from the lowest precedence to the highest:

- Group variables defined in the inventory

- Group variables defined in files in a group_vars subdirectory in the same directory as the inventory or the playbook

- Host variables defined in the inventory

- Host variables defined in files in a host_vars subdirectory in the same directory as the inventory or the playbook

- Host facts, discovered at runtime

- Play variables in the playbook (vars and vars_files)

- Task variables

- Extra variables defined on the command line
For example, a variable that is set to affect the all host group is overridden by a variable that has the same name and is set to affect a single host.
One recommended practice is to choose globally unique variable names, so that you do not have to consider precedence rules. However, sometimes you might want to use precedence to cause different hosts or host groups to get different settings than your defaults.

If the same variable name is defined at more than one level, the level with the highest precedence wins. A narrow scope, such as a host variable or a task variable, takes precedence over a wider scope, such as a group variable or a play variable. Variables defined by the inventory are overridden by variables defined by the playbook. Extra variables defined on the command line with the --extra-vars, or -e, option have the highest precedence.

# Variables in Playbooks
Variables play an important role in Ansible Playbooks because they ease the management of variable data in a playbook.
# Defining Variables in Playbooks
When writing plays, you can define your own variables and then invoke those values in a task. For example, you can define a variable named web_package with a value of httpd. A task can then call the variable using the ansible.builtin.dnf module to install the httpd package.

You can define playbook variables in multiple ways. One common method is to place a variable in a vars block at the beginning of a play:
```
- hosts: all
  vars:
    user: joe
    home: /home/joe
```
It is also possible to define playbook variables in external files. In this case, instead of using a vars block in a play in the playbook, you can use the vars_files directive followed by a list of names for external variable files relative to the location of the playbook:
```
- hosts: all
  vars_files:
    - vars/users.yml
```
The playbook variables are then defined in those files in YAML format:
```
user: joe
home: /home/joe
```
# Using Variables in Playbooks
After you have declared variables, you can use the variables in tasks. Variables are referenced by placing the variable name in double braces ({{ }}). Ansible substitutes the variable with its value when the task is executed.
```
vars:
  user: joe

tasks:
  # This line will read: Creates the user joe
  - name: Creates the user {{ user }}
    user:
      # This line will create the user named Joe
      name: "{{ user }}"
```
>  [!Important]
    When a variable is used as the first element to start a value, quotes are mandatory. This prevents Ansible from interpreting the variable reference as starting a YAML dictionary. 
    The following message appears if quotes are missing:
 ```
ansible.builtin.dnf:
     name: {{ service }}
            ^ here
We could be wrong, but this one looks like it might be an issue with
missing quotes.  Always quote template expression brackets when they
start a value. For instance:

    with_items:
      - {{ foo }}

Should be written as:

    with_items:
      - "{{ foo }}"
```
# Host Variables and Group Variables
Inventory variables that apply directly to hosts fall into two broad categories: host variables apply to a specific host, and group variables apply to all hosts in a host group or in a group of host groups. Host variables take precedence over group variables, but variables defined by a playbook take precedence over both.

One way to define host variables and group variables is to do it directly in the inventory file.
>  [!NOTE]
> This is an earlier approach to defining host and group variables, but you might see it used because it puts all the inventory information and variable settings for hosts and host groups in one file.

Defining the ansible_user host variable for demo.example.com:
```
[servers]
demo.example.com  ansible_user=joe
```
Defining the user group variable for the servers host group.
```
[servers]
demo1.example.com
demo2.example.com

[servers:vars]
user=joe
```
Defining the user group variable for the servers group, which consists of two host groups each with two servers.
```
[servers1]
demo1.example.com
demo2.example.com

[servers2]
demo3.example.com
demo4.example.com

[servers:children]
servers1
servers2

[servers:vars]
user=joe

```
Some disadvantages of this approach are that it makes the inventory file more difficult to work with, it mixes information about hosts and variables in the same file, and it uses an obsolete syntax.

# Using Directories to Populate Host and Group Variables
You can define variables for hosts and host groups by creating two directories, group_vars and host_vars, in the same working directory as the inventory file or playbook. These directories contain files defining group variables and host variables, respectively.
>  [!Important]
> The recommended practice is to define inventory variables using host_vars and group_vars directories, and not to define them directly in the inventory files.


To define group variables for the servers group, you would create a YAML file named group_vars/servers, and then the contents of that file would set variables to values using the same syntax as in a playbook:
```
user: joe
```
Likewise, to define host variables for a particular host, create a file with a name matching the host in the host_vars directory to contain the host variables.

The following examples illustrate this approach in more detail. Consider a scenario where you need to manage two data centers, and the data center hosts are defined in the ~/project/inventory inventory file:
```
[datacenter1]
demo1.example.com
demo2.example.com

[datacenter2]
demo3.example.com
demo4.example.com

[datacenters:children]
datacenter1
datacenter2
```
If you need to define a general value for all servers in both data centers, set a group variable for the datacenters host group:
```
[admin@station project]$ cat ~/project/group_vars/datacenters
package: httpd
```
If you need to define difference values for each data center, set a group variable for each data center host group:
```
[admin@station project]$ cat ~/project/group_vars/datacenter1
package: httpd
[admin@station project]$ cat ~/project/group_vars/datacenter2
package: apache
```
If you need to define different values for each managed host in every data center, then define the variables in separate host variable files:
```
[admin@station project]$ cat ~/project/host_vars/demo1.example.com
package: httpd
[admin@station project]$ cat ~/project/host_vars/demo2.example.com
package: apache
[admin@station project]$ cat ~/project/host_vars/demo3.example.com
package: mariadb-server
[admin@station project]$ cat ~/project/host_vars/demo4.example.com
package: mysql-server
```
The directory structure for the example project, project, if it contained all the example files above, would appear as follows:
```
project
├── ansible.cfg
├── group_vars
│   ├── datacenters
│   ├── datacenters1
│   └── datacenters2
├── host_vars
│   ├── demo1.example.com
│   ├── demo2.example.com
│   ├── demo3.example.com
│   └── demo4.example.com
├── inventory
└── playbook.yml
```
> [!NOTE]
> Ansible looks for host_vars and group_vars subdirectories relative to both the inventory file and the playbook file.
If your inventory and your playbook happen to be in the same directory, this is simple and Ansible looks in that directory for those subdirectories. If your inventory and your playbook are in separate directories, then Ansible looks in both places for host_vars and group_vars subdirectories. The playbook subdirectories have higher precedence.

# Overriding Variables from the Command Line
Inventory variables are overridden by variables set in a playbook, but both kinds of variables can be overridden through arguments passed to the ansible-navigator run command on the command line. Variables set on the command line are called extra variables.

Extra variables can be useful when you need to override the defined value for a variable for a one-off run of a playbook. For example:
```
 ansible-navigator run main.yml -e "package=apache"
```
# Using Dictionaries as Variables
Instead of assigning configuration data that relates to the same element to multiple variables, administrators can use dictionaries. A dictionary is a data structure containing key-value pairs, where the values can also be dictionaries.

For example, consider the following snippet:
user1_first_name: Bob
user1_last_name: Jones
user1_home_dir: /users/bjones
user2_first_name: Anne
user2_last_name: Cook
user2_home_dir: /users/acook
This could be rewritten as a dictionary called users:
```
users:
  bjones:
    first_name: Bob
    last_name: Jones
    home_dir: /users/bjones
  acook:
    first_name: Anne
    last_name: Cook
    home_dir: /users/acook
```
You can then use the following variables to access user data:
```
# Returns 'Bob'
users.bjones.first_name

# Returns '/users/acook'
users.acook.home_dir
```
Because the variable is defined as a Python dictionary, an alternative syntax is available.
```
# Returns 'Bob'
users['bjones']['first_name']

# Returns '/users/acook'
users['acook']['home_dir']
```
>  [!Important]
> The dot notation can cause problems if the key names are the same as names of Python methods or attributes, such as discard, copy, add, and so on. Using the brackets notation can 
  help avoid conflicts and errors.
  Both syntaxes are valid, but to make troubleshooting easier, Red Hat recommends that you use one syntax consistently in all files throughout any given Ansible project.
# Capturing Command Output with Registered Variables
You can use the register statement to capture the output of a command or other information about the execution of a module. The output is saved into a variable that can be used later in the playbook for either debugging purposes or to achieve something else, such as applying a particular configuration setting based on a command's output.

The following play demonstrates how to capture the output of a command for debugging purposes:
```
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
    - name: Install the package
      ansible.builtin.dnf:
        name: httpd
        state: installed
      register: install_result

    - debug:
        var: install_result
```
When you run the play, the debug module dumps the value of the install_result registered variable to the terminal.
```
[user@demo ~]$ ansible-navigator run playbook.yml -m stdout
PLAY [Installs a package and prints the result] ****************************

TASK [setup] ***************************************************************
ok: [demo.example.com]

TASK [Install the package] *************************************************
ok: [demo.example.com]

TASK [debug] ***************************************************************
ok: [demo.example.com] => {
    "install_result": {
        "changed": false,
        "msg": "",
        "rc": 0,
        "results": [
            "httpd-2.4.51-7.el9_0.x86_64 providing httpd is already installed"
        ]
    }
}

PLAY RECAP *****************************************************************
demo.example.com    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# Managing Secrets
 - Objectives
    - Encrypt sensitive variables by using Ansible Vault, and run playbooks that reference Vault-encrypted variable files.
   
  # Introducing Ansible Vault
  Ansible might need access to sensitive data, such as passwords or API keys, to configure managed hosts. Normally, this information is stored as plain text in inventory variables or 
  other Ansible files. In that case, however, any user with access to the Ansible files, or a version control system that stores the Ansible files, would have access to this sensitive 
  data. This poses an obvious security risk.

  Ansible Vault, which is included with Ansible, can be used to encrypt and decrypt any data file used by Ansible. To use Ansible Vault, use the command-line tool named ansible-vault 
  to create, edit, encrypt, decrypt, and view files.
  
  Ansible Vault can encrypt any data file used by Ansible. This might include inventory variables, included variable files in a playbook, variable files passed as arguments when 
  executing the playbook, or variables defined in Ansible roles.

 >  [!Important]
> Ansible Vault does not implement its own cryptographic functions but rather uses an external Python toolkit. Files are protected with symmetric encryption using AES256 with a 
  password as the secret key. Note that the way this is done has not been formally audited by a third party.

