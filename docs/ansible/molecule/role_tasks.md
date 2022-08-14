# Start
Our folder should look something like this

```BASH
$ tree
├── defaults
│   └── main.yml
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── molecule
│   ├── config.yml
│   └── default
│       ├── converge.yml
│       ├── molecule.yml
│       └── verify.yml
├── tasks
│   └── main.yml
├── templates
│   ├── chrony.conf.j2
│   └── ntp.conf.j2
└── vars
│   └── default.yml
├── LICENSE
├── README.md
```

and if we run `molecule test` everything should succed.

## Prepare

The role already has tests, but we want to write new ones, so we delete the old ones

```bash
rm molecule/defualt/verify.yml
```

The main reason we use molecule for testing is thats its very easy to write tests and keep them up to date.  
Normally we follow a `Test-driven development (TDD)` inspired idea and write a test first and then write the needed tasks to archieve the status we defined in our tests.

## What wo we test

Short answer:  `the least possible amount`.  
lLonger answer: We test everything that could potentially fail.

In the ntp role we have, these are the thing i think could possibly fail:

### Installed packages

Um zu überprüfen ob ein Paket installiert ist nehmen wir zB folgenden Code:


```yaml
- name: Check package is installable from repository
  ansible.builtin.package:
    name: "ntp"
    state: present
  register: package_installed
  failed_when: (package_installed is changed) or (package_installed is failed)
```
We check if the ntp package is installed.  

But the ntp package has different names on different platforms, so wie add something to the tests.

```yaml
- name: CentOS/RedHat >=8 | Check if chrony is installed
    package:
    name: "chrony"
    state: present
    check_mode: true
    register: ntp_installed
    failed_when: (ntp_installed is changed) or (ntp_installed is failed)
    when:
    - ansible_distribution == "CentOS"
    - ansible_distribution_major_version >= "8"

- name: Check if ntp is installed
    package:
    name: "ntp"
    state: present
    check_mode: true
    register: ntp_installed
    failed_when: (ntp_installed is changed) or (ntp_installed is failed)
    when:
    - (ansible_distribution == "CentOS" and ansible_distribution_major_version < "8") or
        ansible_os_family == "Debian"
```

Now we test for the package `chrony`instead of the package `ntp`on everything that is RedHat based and has a version greate or equal 8 (Centos Stream 8/Centos Stream 9/Almalinux 8 and 9 or Rocky Linuy 8 and 9).

### Running services

we check if the service is running

```yaml
- name: check ntp daemon
  service:
    name: "{{ ntp_daemon }}"
    state: started
    enabled: true
  check_mode: true
  register: molecule_service
  failed_when: (molecule_service is changed) or (molecule_service is failed)
```

And here qe have the same problem as the package

```yaml
    - name: CentOS/RedHat >=8 | check chronyd daemon
      service:
        name: "chronyd"
        state: started
        enabled: true
      check_mode: true
      register: chronyd_service
      failed_when: (chronyd_service is changed) or (chronyd_service is failed)
      when:
        - ansible_distribution == "CentOS"
        - ansible_distribution_major_version >= "8"

    - name: CentOS/RedHat <8 | check ntpd daemon
      service:
        name: "ntpd"
        state: started
        enabled: true
      check_mode: true
      register: ntpd_service
      failed_when: (ntpd_service is changed) or (ntpd_service is failed)
      when:
        - (ansible_distribution == "CentOS" and ansible_distribution_major_version < "8")

    - name: check ntp daemon
      service:
        name: "ntp"
        state: started
        enabled: true
      check_mode: true
      register: ntpd_service
      failed_when: (ntpd_service is changed) or (ntpd_service is failed)
      when:
        - ansible_os_family == "Debian"
```

Here we hava a difference between  CentOS-Stream 8/9, CentOS 7 and Debian/Ubuntu-based systems.

### Erstellte Konfigurationen

We check the configuration file ntp.conf/chrony.conf.

```yaml
- name: Test if /etc/chrony.conf exists
    lineinfile:
    name: '/etc/chrony.conf'
    line: 'pool 0.de.pool.ntp.org iburst'
    state: present
    check_mode: true
    register: ntp_conf
    failed_when: (ntp_conf is changed) or (ntp_conf is failed)
```

and ans usual the split version

```yaml
- name: CentOS/RedHat >=8 | Test if /etc/chrony.conf exists
    lineinfile:
    name: '/etc/chrony.conf'
    line: 'pool 0.de.pool.ntp.org iburst'
    state: present
    check_mode: true
    register: chrony_conf
    failed_when: (chrony_conf is changed) or (chrony_conf is failed)
    when:
    - ansible_distribution == "CentOS"
    - ansible_distribution_major_version >= "8"

- name: Test if /etc/ntp.conf exists
    lineinfile:
    name: '/etc/ntp.conf'
    line: 'server 0.de.pool.ntp.org iburst'
    state: present
    check_mode: true
    register: ntp_conf
    failed_when: (ntp_conf is changed) or (ntp_conf is failed)
    when:
    - (ansible_distribution == "CentOS" and ansible_distribution_major_version < "8") or
        ansible_os_family == "Debian"
```

## Verify

Now we have new tests and can run `molecule verify` again with hopefully the same result.
