
# Snippets

Here is a collection of some snippets for my `verify.yml` from my testing with molecule.

## Packages

### Check if package is already installed
```yaml
- name: Check package is installed
  ansible.builtin.package:
    name: "PACKAGENAME"
    state: present
  register: verify_package_installed
  failed_when: (verify_package_installed is changed) or (verify_package_installed is failed)
```

### Check if package is installable (e.g. after adding a new repository)
```yaml
- name: Check package is installable from repository
  ansible.builtin.package:
    name: "PACKAGENAME_FROM_REPOSITORY"
    state: present
  register: verify_package_installable
```


## User/Groups

### verify group

```yaml
- name: verify if group exists (default)
  ansible.builtin.group:
    name: "GROUPNAME"
  check_mode: true
  register: verify_group
  failed_when: (verify_group is changed) or (verify_group is failed)

- name: verify if group exists (full_opts)
  ansible.builtin.group:
    name: "GROUPNAME"
    gid: "4711"
    state: "present"
    system: "true"
  check_mode: true
  register: verify_group
  failed_when: (verify_group is changed) or (verify_group is failed)

- name: verify if group is absent
  ansible.builtin.group:
    name: "ABSENT_GROUP"
    state: "absent"
  check_mode: true
  register: verify_group_absent
  failed_when: (verify_group_absent is changed) or (verify_group_absent is failed)
```


## Files/Folders

### Test if file exists and contains text

```yaml
- name: verify if file exists and contains text
  ansible.builtin.lineinfile:
    name: '/PATH/TO/FILE'
    line: 'SEARCHTEXT'
    state: present
  check_mode: true
  register: verify_file_content
  failed_when: (verify_file_content is changed) or (verify_file_content is failed)
```

### verify file/mode

```yaml
- name: verify if filemode
  file:
    name: '/PATH/TO/FILE'
    owner: "root"
    group: "root"
    mode: "0700"
  register: verify_file_mode
  failed_when: (verify_file_mode is changed) or (verify_file_mode is failed)
```

## Daemons/Services

### Verify service is running and enabled

```yaml
- name: check service
  service:
    name: "SERVICENAME"
    state: started
    enabled: true
  check_mode: true
  register: verify_service
  failed_when: (verify_service is changed) or (verify_service is failed)
```

## Webservices

## Helper

### OS Mappings

```yaml
os_repo_file:
  EL:
    "7": "CentOS-Linux-PowerTools.repo"
    "8": "CentOS-Stream-PowerTools.repo"
  CentOS:
    "7": "CentOS-Linux-PowerTools.repo"
    "8": "CentOS-Stream-PowerTools.repo"
    "9": "CentOS-Stream-PowerTools.repo"
  Rocky: "CentOS-Stream-PowerTools.repo"
  AlmaLinux: "CentOS-Stream-PowerTools.repo"
---
repo_file: '/etc/yum.repos.d/{{ os_repo_file[ansible_distribution][ansible_distribution_major_version] | os_repo_file[ansible_distribution] }}'
```

### OS Filter

```YAML
when:
  - (ansible_distribution == "RedHat" and ansible_distribution_major_version >= "8") or
    (ansible_distribution == "Fedora") or
    (ansible_distribution == "Rocky") or
    (ansible_distribution == "AlmaLinux")
```

## Other

### verify binarys

```yaml
- name: verify binary exists
  command: '/PATH/TO/BINARY'
  register: verify_binary
  failed_when: not verify_binary.stat.exists
```

### verify binary/script executes without error

```yaml
- name: verify binary exists
  command: '/PATH/TO/BINARY version'
  register: verify_script_execution
  failed_when: verify_script_execution.rc != 0
```

### verify cron

```yaml
- name: verify cron
  cron:
    name: "CRONNAME"
    minute: "5"
    hour: "*"
    day: "*"
    weekday: "*"
    month: "*"
    job: "/PATH/TO/CRON/FILE"
  register: verify_cron
  failed_when: (verify_cron is changed) or (verify_cron is failed)
```
