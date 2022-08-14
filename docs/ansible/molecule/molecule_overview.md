# Overview files and folders

As for this Tutorial we will create a new role from scratch. I will use my [ansible generator](https://github.com/mullholland/ansible-generator) to automate some of the mundane things away.

First i assume you have an empty github repository with the name of the role. In this example i will use `ansible-role-ntp` in this tutorial.

```BASH
cd ansible-role-ntp
# execute the playbook from ansible-generator to create an empty role
/PATH/TO/REPOSITORY/ansible-generator/role-generator/generate_empty.yml
```

This will create the following folders/files.

```BASH
$ tree -a
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── molecule
│   ├── config.yml
│   └── default
│       ├── converge.yml
│       ├── molecule.yml
│       ├── prepare.yml
│       ├── requirements.yml
│       └── verify.yml
├── tasks
│   └── main.yml
├── templates
├── vars
│   └── default.yml
├── .gitignore
├── .editorconfig
├── .ansible-lint
└── .yamllint
```

It is important do differentiate between the folder `molecule/` and all the other folders and files.

Now lets take a closer look at the fodlers and the files in them.
If you want more in depth details, you can always take a look at the [official documentation](https://galaxy.ansible.com/docs/contributing/creating_role.html)

## defaults/

In the `main.yml` are all the role default variables.  
These variables are easy to overide inside a play.
If you use the [ansible generator](https://github.com/mullholland/ansible-generator) here is also the place to create a part of your documentation. Everything in the `defaults/main.yml` will also be in the README.md

## files/

Here you can save static files for the Role. If you need templated files, these are stored in `templates/`.

## handlers/

Here are all handlers for you role. This is mostly used for restarting a service after config change, but can be anythin you want.  
Handlers are mostly executed at the end of the playbook run, so plan for that accordingly.

## meta/

Here is the meta information of your role. The information here ist mostly used in [ansible galaxy](https://galaxy.ansible.com/). But molecule and the [ansible generator](https://github.com/mullholland/ansible-generator) also use some of it. On creation this file will contain.

```yaml
  author: {{ author }}
  namespace: {{ github_namespace }}
  role_name: {{ role_name }}
```

and some more defaults.

## tasks/
This is the base of the role, where all the magix happens.
Here you can write all your tasks that are needed for your role.

## templates/
Here are mostly the jinja2 tempaltes for your role like configuration files or similar files.

## vars/
Here you can save variables which are typically not changed by the user, like the webserver name in RedHat/Debian `http` or `apache2`
This variables have a higher priority than the variables under `defaults/main` in the [ansible precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable).
You will see that i do not use a `main.yml` here. The `main.yml` you be read during the initialisation of the role for all hosts. Put variables here that are the same for all hosts and are typically not changed.  
My more common usage is to put variables with a dependency here. Like the default task from [ansible generator](https://github.com/mullholland/ansible-generator) in the `tasks/main.yml`

```yaml
- name: Include distribution specific variables
  ansible.builtin.include_vars: "{{ item }}"
  with_first_found:
    - '{{ ansible_distribution }}-{{ ansible_distribution_major_version }}.yml'
    - '{{ ansible_distribution }}.yml'
    - '{{ ansible_os_family }}.yml'
    - '{{ ansible_system }}.yml'
    - 'default.yml'
```
This task uses the gathered facts of the hosts, and includes a variable file according to thes facts and the available files. If no specific file is found it includes the `vars/default.yml`.
with a missing `vars/main.yml` and this task you can make sure that only varaiables that you want are imported, if the `vars/defaults.yml` would be named `vars/main.yml` you would always include those variables if wou want it or not.


## single files

Here we take a look at some of the single files in this repository. These are not need by ansible, but they are commonly used by me.
#### .gitignore
You should always have a `.gitignore` file.
If you do not want to write one yourself you can use a service like [gitignore.io](https://gitignore.io). In this file the commonly used tmp files for python,intellij,ansible,terraform,vim and linux are ignored.

#### .editorconfig
I always recommend [Editorconfig](https://editorconfig.org/) because it helps maintain consistent coding styles.

#### .ansible-lint
My individual config file for `ansible-lint`.

#### .yamllint
My individual config file for `yamllint`.

## molecule/
Last but not least, lets take a look at the `molecule/`folder and its files.

#### config.yml
This is no default `molceule` file, but a config file the [ansible generator](https://github.com/mullholland/ansible-generator) uses to determine which [molecule scenarios](https://molecule.readthedocs.io/en/latest/getting-started.html#molecule-scenarios) exists and on which OS each scenario should be tested. With this information the `.github/workflows` and `.gitlab-yi.yml` are created.

```YAML
# molecule_exceptions:
#   - variation: ubuntu2204
#     reason: "Not yet released"
molecule_scenarios:
  - scenario: "default"
    ansible: ["previous", "current", "development"]
    platforms:
      - "ubuntu1804"
      - "ubuntu2004"
      - "centos7"
      - "centos-stream8"
      - "centos-stream9"
      - "ubi8"
#  - scenario: "alternative"
#    ansible: ["previous", "current", "development"]
#    platforms: ["ubuntu1604", "ubuntu1804", "ubuntu2004", "centos7"]
```

The `molecule_exceptions`are used in the `README.md` to document what was not tested.

#### molecule/default/
The folder names in the `molecule/` define the scenario name and the default scenario is named `default`. If you have no special use cases like clustering or so, you do not have to change anything here.  
If you need a new scenario, just copy the folder and renamed it to an appropriate name for your new scenario and dont forget to configure you files inside accordingly.
For the usage with  the [ansible generator](https://github.com/mullholland/ansible-generator) you should also update the `molceule/config.yml` and add your scenario there.

##### converge.yml
The `converge.yml` is the playbook that is executed when you use molecule. In here should be everything yo  eed to execute your role successfully. Idealy most of it ist already in the vars files `defaults/main.yml` or in the `vars/` files.

##### molecule.yml
This is the control file for all you molecule tests. Typically you do not need to change anything here. Changes are mostly needed when you add a scenario, need more than 1 container for your test.
Take a look at this file to understnad what is in there and change it ro accommodate your needs.

##### prepare.yml
If you role needs any preperation, which you do not want to handle inside the role itself, for example your backup script should not install a cron deamon or you manage your repositories in extra roles outside of your application roles, you can do this here.
Anything that happens in this play is not taken into account on the idempotence tasks.

##### requirements.yml
Here you can add roles or collections you need for your preperation. Because ansible galaxy is normally a bit picky, we added the following to the `molecule.yml`

```yaml
dependency:
  name: galaxy
  options:
    role-file: molecule/default/requirements.yml
    requirements-file: molecule/default/requirements.yml
```

with this roles and collections can be in the same file, without this they would each need their own file.

##### verify.yml
Here you write your tests. These will be executed
Hier findet das Testen der Rolle statt. Alle Tests werden hier direkt in Ansible geschrieben, was es stark vereinfacht viele Tests zu schreiben. 
Der Grundsatz hier ist das die Tests nicht von der Rolle wissen, sondern nur wissen wie das ergebniss aussehen soll, daher teilen die Tests und die Rolle keine Variablen, damit Fehler geworfen werden, wenn sich eins von beidem ändert.
