# Install

We want to install everything we need for an working [Ansible](https://www.ansible.com/) / [Molecule](https://molecule.readthedocs.io/en/latest/) environment to create [Ansible roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) and test them with Molecule.

I will try to show different installationmethods that i am using (there are probably more ways to do what you want) and summarize some pros and cons.

Then we will create a little role and write the needed tests for it.

## Requirements

For all of our tests we need a working [Docker](https://docs.docker.com/get-docker/) environment, to test the roles. How you to install Docker is documented [here](https://docs.docker.com/get-docker/). If you do not know what docker is, take a look [here](https://docs.docker.com/get-started/).  
And a basic understanding of Ansible.

## Install OS Packages

A simple way to install Ansible is to use simple OS packages like
```bash
# Debian/Ubuntu
apt-get install ansible

# RedHat/CentOS
yum/dnf install ansible
```

With this you can go pretty far in using Ansible alone, but you migth not always get the newest version of addon you are looking for.  
You could probably combine this method with the next method, but you would probably run into some problems along the road.

## Install PIP

[PIP](https://pypi.org/project/pip/) is a python package installer, which allows you to simply install the oython packages you need.

### OS PIP

To simply use pip and install python packages for your system python use:
```bash
# Upgrade PIP
python -m pip install --upgrade pip
# Install ansible/molecule packages in virtualenvironment
pip3 install  \
  --no-cache-dir  \
  --upgrade  \
  --no-compile \
  ansible \
  requests \
  netaddr \
  molecule[docker,ansible,lint] \
  pytest-testinfra \
  yamllint \
  ansible-lint \
  flake8 \
  pyyaml
```

The first part updates the current pip version, which if often out-of-date, and the next command installs a couple of python packages which we need to test our Ansible roles.

With this we can now begin.

### VENV PIP

If you are unsure with the images, you do not want to mess anything up or experiment with multiple Ansible versions, you should use a [python virtual environment](https://docs.python.org/3/tutorial/venv.html).  
Long story short, theses virtual environments are independent from your system python and you are less likely to mess things up and can experiment with different packages.  

```bash
# Create virtualenvironment
python3 -m venv "${HOME}/.virtualenvs/molecule"
# "enable" virtualenvironment
source "${HOME}/.virtualenvs/molecule/bin/activate"
# Upgrade PIP
python -m pip install --upgrade pip
# Install ansible/molecule packages in virtualenvironment
pip3 install  \
  --no-cache-dir  \
  --upgrade  \
  --no-compile \
  ansible \
  requests \
  netaddr \
  molecule[docker,ansible,lint] \
  pytest-testinfra \
  yamllint \
  ansible-lint \
  flake8 \
  pyyaml
```

The first command creates the python virtual environment in your `HOME/.virtualenvs/molecule`. This python environemnt is completly independent from you OS python.

The second command `activates` your environment to be used instead of the OS python. You can usually see if the environment python is active if there is a `(molecule)` in your shell.

The rest installs all the same packages like in the OS install, but they are now confined in your python virtual environment.

With this we can now begin.

## Nested Docker

An even easier version can be to run everything in docker. So you do not have the think about anything to install besides docker, but you need this anyway.

For this i currently use [this image](https://github.com/mullholland/docker-molecule-shell), which is the same that my roles use for testing.
This enables me to be relative consistent in my testing, if it works on my machine, it will probably work while the automated tests are running.

You can take a look in the [Dockerfile](https://github.com/mullholland/docker-molecule-shell/blob/main/Dockerfile) what is happening in the container.  
The container is based on an ubuntu 22.04, installs everything it need for systemd (not strictly needed) and installs everythin you need for ansible. I create 2 Python virtual environments (Line 78 and Lone 84) which are called `previous` and `current`. These are used in my roles to test the roles against different Ansible version. I used to have a development branch in there, but there were very frequent problems, not with the roles, but witht the dependencies, because `ansible`, `ansible-core` and other packages are developing at an different rate and accessibility. So the to reduce the amount of dependency fixes i have to do, is just skipped it ;).

You can pull the image and follow the guide in the [README](https://github.com/mullholland/docker-molecule-shell/blob/main/README.md)