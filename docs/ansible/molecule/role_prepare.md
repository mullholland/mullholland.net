# Role - Prepare

## cleanup
For this tutorial we concentrate on writing a couple of tests and not on writing the role.
We copy the content from this [my ntp role](https://github.com/mullholland/ansible-role-ntp) inside the folder and skip to the interesting part.

## molecule - create

Creates the desired container for us. From here we cann also run a quick `molecule converge`, this should give us an output like this:

```bash
PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [default-current-ubuntu2004]

TASK [vim : Include distribution specific variables] ***************************
ok: [default-current-ubuntu2004] => (item=/home/mullholland/Downloads/tmp/ntp/vars/default.yml)

PLAY RECAP *********************************************************************
default-current-ubuntu2004 : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Here we see, we do not have any task in the play, except the default variable include the Ansibel generator adds.

Now we can begin.
