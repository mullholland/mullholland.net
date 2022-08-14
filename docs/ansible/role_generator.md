# Role generator

## Simplified workflow

``` mermaid
graph TD
  A[New Role] --> B[generate_empty] --> C[writing ansible role];
  C --> D[molecule create];
  D --> E[molecule converge];
  E -->|Everything works| F[molecule idempotence];
  E -->|Not everything works| C;
  F -->|Everything works| G[molecule verify];
  F -->|Not everything works| C;
  G -->|Everything works| H[generate_docs];
  G -->|Not everything works| C;
  H --> I[git commit and push];
  I --> K[github actions];
  J[New Feature] --> C[writing ansible role];
  K -->|Not everything works| C;
  K -->|Everything works| L[Release to Ansible Galaxy];
```

## Usage

Here is an overview for the usage in creating a new role with the [role generator](https://github.com/mullholland/ansible-generator) and updating an existing role with ne documentation and CI.

### customize

You should take a look at the variable in `vars/main.yml` to customize the output.

### New role

To create the templates for a new and empty role. Change your working directory to the empty repository and execute the following:

```bash
# we assume that our empty role will be a role to install vim
mkdir ansible-role-vim
cd ansible-role-vim
/PATH/TO/REPOSITORY/ansible-generator/role-generator/generate_empty.yml
```

This will create the following files and folders

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

A detailed explanation of the files and folders can be found [here](role_generator_new.md)

### Updating a role

Updating a role with the `role-generator` is split in three parts.  

1. Make changes to the role as you need from  point-of-view of ansible/molecule, but ignore the CI files and README.md.
2. Execute the `/PATH/TO/REPOSITORY/ansible-generator/role-generator/generate_docs.yml`, this will update the CI files and the README.md
3. commit your changes.
