# auditbeat

[![Source Code](https://img.shields.io/badge/github-source%20code-blue?logo=github&logoColor=white)](https://github.com/rolehippie/auditbeat) [![Testing Build](https://github.com/rolehippie/auditbeat/workflows/testing/badge.svg)](https://github.com/rolehippie/auditbeat/actions?query=workflow%3Atesting) [![Readme Build](https://github.com/rolehippie/auditbeat/workflows/readme/badge.svg)](https://github.com/rolehippie/auditbeat/actions?query=workflow%3Areadme) [![Galaxy Build](https://github.com/rolehippie/auditbeat/workflows/galaxy/badge.svg)](https://github.com/rolehippie/auditbeat/actions?query=workflow%3Agalaxy) [![License: Apache-2.0](https://img.shields.io/github/license/rolehippie/auditbeat)](https://github.com/rolehippie/auditbeat/blob/master/LICENSE)

Ansible role to install and configure auditbeat.

## Sponsor

Building and improving this Ansible role have been sponsored by my current and previous employers like **[Cloudpunks GmbH](https://cloudpunks.de)** and **[Proact Deutschland GmbH](https://www.proact.eu)**.

## Table of content

- [Default Variables](#default-variables)
  - [auditbeat_console_enabled](#auditbeat_console_enabled)
  - [auditbeat_default_modules](#auditbeat_default_modules)
  - [auditbeat_default_processors](#auditbeat_default_processors)
  - [auditbeat_default_rules](#auditbeat_default_rules)
  - [auditbeat_group_modules](#auditbeat_group_modules)
  - [auditbeat_group_processors](#auditbeat_group_processors)
  - [auditbeat_group_rules](#auditbeat_group_rules)
  - [auditbeat_host_modules](#auditbeat_host_modules)
  - [auditbeat_host_processors](#auditbeat_host_processors)
  - [auditbeat_host_rules](#auditbeat_host_rules)
  - [auditbeat_logging_level](#auditbeat_logging_level)
  - [auditbeat_logging_selectors](#auditbeat_logging_selectors)
  - [auditbeat_logstash_enabled](#auditbeat_logstash_enabled)
  - [auditbeat_logstash_hosts](#auditbeat_logstash_hosts)
  - [auditbeat_major_version](#auditbeat_major_version)
  - [auditbeat_name](#auditbeat_name)
  - [auditbeat_rules_path](#auditbeat_rules_path)
  - [auditbeat_service_enabled](#auditbeat_service_enabled)
  - [auditbeat_suid_guid_rule_enabled](#auditbeat_suid_guid_rule_enabled)
  - [auditbeat_suid_guid_rule_files](#auditbeat_suid_guid_rule_files)
  - [auditbeat_tags](#auditbeat_tags)
- [Discovered Tags](#discovered-tags)
- [Dependencies](#dependencies)
- [License](#license)
- [Author](#author)

---

## Default Variables

### auditbeat_console_enabled

#### Default value

```YAML
auditbeat_console_enabled: false
```

### auditbeat_default_modules

List of default modules, gets directly transformed to yaml

#### Default value

```YAML
auditbeat_default_modules:
  - module: auditd
    audit_rule_files:
      - '{{ auditbeat_rules_path }}/*.conf'
  - module: file_integrity
    paths:
      - /bin
      - /usr/bin
      - /sbin
      - /usr/sbin
      - /etc
  - module: system
    datasets:
      - package
    period: 10m
  - module: system
    datasets:
      - host
      - login
      - process
      - socket
      - user
    state.period: 12h
    user.detect_password_changes: true
    login.wtmp_file_pattern: /var/log/wtmp*
    login.btmp_file_pattern: /var/log/btmp*
```

### auditbeat_default_processors

List of default processors, gets directly transformed to yaml

#### Default value

```YAML
auditbeat_default_processors:
  - add_host_metadata:
  - add_cloud_metadata:
  - add_docker_metadata:
```

### auditbeat_default_rules

List of default rules, gets written to separate files

#### Default value

```YAML
auditbeat_default_rules:
  - name: current-dir
    comment: Ignore current working directory records
    rule:
      - -a always,exclude -F msgtype=CWD
  - name: ignore-eoe
    comment: Ignore EOE records (End Of Event, not needed)
    rule:
      - -a always,exclude -F msgtype=EOE
  - name: high-volume
    comment: High Volume Event Filter
    rule:
      - -a exit,never -F arch=b32 -F dir=/dev/shm -k sharedmemaccess
      - -a exit,never -F arch=b64 -F dir=/dev/shm -k sharedmemaccess
      - -a exit,never -F arch=b32 -F dir=/var/lock/lvm -k locklvm
      - -a exit,never -F arch=b64 -F dir=/var/lock/lvm -k locklvm
  - name: ignore-cron
    comment: Cron jobs fill the logs useless stuff
    rule:
      - -a never,user -F subj_type=crond_t
      - -a exit,never -F subj_type=crond_t
  - name: cis-4_1_4
    weight: 20
    comment: CIS 4.1.4 - Changes to the time
    rule:
      - -a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
      - -a always,exit -F arch=b32 -S adjtimex -S settimeofday -S stime -k time-change
      - -a always,exit -F arch=b64 -S clock_settime -k time-change
      - -a always,exit -F arch=b32 -S clock_settime -k time-change
      - -w /etc/localtime -p wa -k time-change
  - name: cis-4_1_5
    weight: 20
    comment: CIS 4.1.5 - Changes to user/group information
    rule:
      - -w /etc/group -p wa -k identity
      - -w /etc/passwd -p wa -k identity
      - -w /etc/gshadow -p wa -k identity
      - -w /etc/shadow -p wa -k identity
      - -w /etc/security/opasswd -p wa -k identity
  - name: cis-4_1_6
    weight: 20
    comment: CIS 4.1.6 - Changes to the network environment
    rule:
      - -a exit,always -F arch=b64 -S sethostname -S setdomainname -k system-locale
      - -a exit,always -F arch=b32 -S sethostname -S setdomainname -k system-locale
      - -w /etc/issue -p wa -k system-locale
      - -w /etc/issue.net -p wa -k system-locale
      - -w /etc/hosts -p wa -k system-locale
      - -w /etc/network -p wa -k system-locale
  - name: cis-4_1_7
    weight: 20
    comment: CIS 4.1.7 - Changes to system's Mandatory Access Controls
    rule:
      - -w /etc/apparmor/ -p wa -k MAC-policy
      - -w /etc/apparmor.d/ -p wa -k MAC-policy
  - name: cis-4_1_8
    weight: 20
    comment: CIS 4.1.8 - Log login/logout events
    rule:
      - -w /var/log/faillog -p wa -k logins
      - -w /var/log/lastlog -p wa -k logins
      - -w /var/log/tallylog -p wa -k logins
  - name: cis-4_1_9
    weight: 20
    comment: CIS 4.1.9 - Log session initiation information
    rule:
      - -w /var/run/utmp -p wa -k session
      - -w /var/log/wtmp -p wa -k logins
      - -w /var/log/btmp -p wa -k logins
  - name: cis-4_1_10
    weight: 20
    comment: CIS 4.1.10 - Log Discretionary Access Control modifications
    rule:
      - -a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F
        auid!=4294967295 -k perm_mod
      - -a always,exit -F arch=b32 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F
        auid!=4294967295 -k perm_mod
      - -a always,exit -F arch=b64 -S chown -S fchown -S fchownat -S lchown -F auid>=1000
        -F auid!=4294967295 -k perm_mod
      - -a always,exit -F arch=b32 -S chown -S fchown -S fchownat -S lchown -F auid>=1000
        -F auid!=4294967295 -k perm_mod
      - -a always,exit -F arch=b64 -S setxattr -S lsetxattr -S fsetxattr -S removexattr
        -S lremovexattr -S fremovexattr -F auid>=1000 -F auid!=4294967295 -k perm_mod
      - -a always,exit -F arch=b32 -S setxattr -S lsetxattr -S fsetxattr -S removexattr
        -S lremovexattr -S fremovexattr -F auid>=1000 -F auid!=4294967295 -k perm_mod
  - name: cis-4_1_11
    weight: 20
    comment: CIS 4.1.11 - Log unsuccessful unauthorized file access attempts
    rule:
      - -a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -S ftruncate
        -F exit=-EACCES -F auid>=1000 -F auid!=4294967295 -k access
      - -a always,exit -F arch=b32 -S creat -S open -S openat -S truncate -S ftruncate
        -F exit=-EACCES -F auid>=1000 -F auid!=4294967295 -k access
      - -a always,exit -F arch=b64 -S creat -S open -S openat -S truncate -S ftruncate
        -F exit=-EPERM -F auid>=1000 -F auid!=4294967295 -k access
      - -a always,exit -F arch=b32 -S creat -S open -S openat -S truncate -S ftruncate
        -F exit=-EPERM -F auid>=1000 -F auid!=4294967295 -k access
  - name: cis-4_1_12
    weight: 20
    comment: CIS 4.1.12 - Log use of privileged commands
    rule: |
      {% for file in auditbeat_suid_guid_rule_files %}
      -a always,exit -F path={{ file }} -F perm=x -F auid>=1000 -F auid!=4294967295 -k privileged
      {% endfor %}
    state: "{{ 'present' if auditbeat_suid_guid_rule_files | length > 0 else 'absent'\
      \ }}"
  - name: cis-4_1_13
    weight: 20
    comment: CIS 4.1.13 - Log successful file system mounts
    rule:
      - -a always,exit -F arch=b64 -S mount -F auid>=1000 -F auid!=4294967295 -k mounts
      - -a always,exit -F arch=b32 -S mount -F auid>=1000 -F auid!=4294967295 -k mounts
  - name: cis-4_1_14
    weight: 20
    comment: CIS 4.1.14 - Log file deletion Events by User
    rule:
      - -a always,exit -F arch=b64 -S unlink -S unlinkat -S rename -S renameat -F
        auid>=1000 -F auid!=4294967295 -k delete
      - -a always,exit -F arch=b32 -S unlink -S unlinkat -S rename -S renameat -F
        auid>=1000 -F auid!=4294967295 -k delete
  - name: cis-4_1_15
    weight: 20
    comment: CIS 4.1.15 - Log changes to sudoers
    rule:
      - -w /etc/sudoers -p wa -k scope
      - -w /etc/sudoers.d/ -p wa -k scope
  - name: cis-4_1_16
    weight: 20
    comment: CIS 4.1.16 - Log sudolog
    rule:
      - -w /var/log/sudo.log -p wa -k actions
  - name: cis-4_1_17
    weight: 20
    comment: CIS 4.1.17 - Log kernel module actions
    rule:
      - -w /sbin/insmod -p x -k modules
      - -w /sbin/rmmod -p x -k modules
      - -w /sbin/modprobe -p x -k modules
      - -a always,exit -F arch=b64 -S init_module -S delete_module -k modules
```

#### Example usage

```YAML
auditbeat_default_rules:
  - name: workingdir
    comment: Ignore current working directory records
    rule: |
      -a always,exclude -F msgtype=CWD
  - name: eventfilter
    comment: High Volume Event Filter
    rule:
      - '-a exit,never -F arch=b32 -F dir=/dev/shm -k sharedmemaccess'
  - name: foobar
    state: absent
```

### auditbeat_group_modules

List of group modules, merged with auditbeat_default_modules

#### Default value

```YAML
auditbeat_group_modules: []
```

### auditbeat_group_processors

List of group processors, merged with auditbeat_default_processors

#### Default value

```YAML
auditbeat_group_processors: []
```

### auditbeat_group_rules

List of group rules, merged with auditbeat_default_rules

#### Default value

```YAML
auditbeat_group_rules: []
```

### auditbeat_host_modules

List of host modules, merged with auditbeat_default_modules

#### Default value

```YAML
auditbeat_host_modules: []
```

### auditbeat_host_processors

List of group processors, merged with auditbeat_default_processors

#### Default value

```YAML
auditbeat_host_processors: []
```

### auditbeat_host_rules

List of host rules, merged with auditbeat_default_rules

#### Default value

```YAML
auditbeat_host_rules: []
```

### auditbeat_logging_level

Define logging level

#### Default value

```YAML
auditbeat_logging_level: info
```

### auditbeat_logging_selectors

Define logging selectors, like beat, publish, service

#### Default value

```YAML
auditbeat_logging_selectors: []
```

### auditbeat_logstash_enabled

#### Default value

```YAML
auditbeat_logstash_enabled: true
```

### auditbeat_logstash_hosts

#### Default value

```YAML
auditbeat_logstash_hosts: []
```

### auditbeat_major_version

Major version to install, used for the APT repository

#### Default value

```YAML
auditbeat_major_version: 7
```

### auditbeat_name

Name of the shipper within the output

#### Default value

```YAML
auditbeat_name: '{{ ansible_hostname }}'
```

### auditbeat_rules_path

Path to store the defined rules

#### Default value

```YAML
auditbeat_rules_path: /etc/auditbeat/audit.rules.d
```

### auditbeat_service_enabled

Enable the console output

### auditbeat_suid_guid_rule_enabled

Search suid/guid programs on disk

#### Default value

```YAML
auditbeat_suid_guid_rule_enabled: false
```

### auditbeat_suid_guid_rule_files

Define a default list of suid/guid files

#### Default value

```YAML
auditbeat_suid_guid_rule_files: []
```

### auditbeat_tags

List of tags to assign for the shipper

#### Default value

```YAML
auditbeat_tags: []
```

## Discovered Tags

**_auditbeat_**


## Dependencies

- None

## License

Apache-2.0

## Author

[Thomas Boerger](https://github.com/tboerger)
