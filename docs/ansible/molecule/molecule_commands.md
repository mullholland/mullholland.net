## Molecule command overview

Here are the molecule commands, which are relevant to our role tests.
Writing the role and testign the role should go hand in hand and you should use the appropriate commands often.

# Test

The `molecule test` command does it all. It creates the container, executes the role und executes the tests and deletes the container as if nothing was ever there.
That is the command that must succed at the end, but should not be use everytime you have a chane in your role.
During local testing we execute only the molecule commands that we need at the time.

### lint

With `molecule lint` you have a very strict, but in my opinion also very important command. `molecule lint` executes the `ansible-lint` and `yamllint` commands and tests your 'style'. It checks your syntax and task definition for best practices. During the `molecule test`command this is the first that happens, if you code is not up to this rules, nothing else will happen.
You should get used to its errors, it really helps to get a nice style in the role.

### create & prepare

# Create

The `molecule create` command is the beginning of every new session to create or change a role.
It creates the in `molecule/default/molecule.yml` defined environment and executes directly `molceule prepare`.

# Prepare

The `molecule prepare` command 'prepares' the container according to our wishes in the file `molecule/default/prepare.yml`. Normally this is used to install dependencies, either other roles or just simple files/programs directly. This happens only on the creation of the container and will not happen again if the container is not deleted.
Everything that happens here ist not subject to the `moelcule idempotence` test.

If you have nothing to 'prepare' and no required roles, you should delete the files `molecule/default/prepare.yml` and `molecule/default/requirements.yml` to avoid errors.

### converge

`Molecule converge` executes your role inside the container. It starts an Ansible play wiht the `molecule/default/converge.yml` and simulates to a default execution of your role. You cann execute this as often as you like. I recommand it after every change of your role to control if you role still works.

If `molecule converge` runs without error, it is a first indication that you role works.

### idempotence

After the role is done and the last step was successfull the command `molecule idempotence` tests the next step. Every Ansible role should be [idempotent](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency) and this commands tests it. It runs the same commands as `molecule converge`and fails if any task is `changed` or `errors`.

### verify

The `molecule verify` is the last step to test and runs the written tests.

### destroy

Execution of `molecule destroy` deletes the container. Mostly used when we are finished or want to begin from scratch. Also the first AND last task in `molecule test`.

### login

With `molecule login` we can log into our container to check on changes ansible should have made. Take a look at file contents or fiel permissions and so on.

## Molecule Tipps & Tricks

### Change OS

The OS of the container we are testing against is managed by the ENV Variable `platform`.
The Default is `ubuntu2004` and can be changed with.

```bash
export platform=centos-stream8
```

You can choose between
```
- ubuntu1604
- ubuntu1804
- ubuntu2004
- ubuntu2204
- centos7
- centos-stream8
- centos-stream9
- ubi8
```

and some more, just look into my [github repositories](https://github.com/mullholland?tab=repositories&q=docker-molecue&type=&language=&sort=).  
Bevor you change the OS you should delete the running container, molecule becomes confused very easily.
