aspects_iptables
=========

Manage iptables using CentOS's built in service and Debian/Ubuntu's iptables-persistent package.

**Warning**: Removes firewalld on CentOS 7 so that the iptables-service package can be used.

Requirements
------------

Set ```hash_behaviour=merge``` in your ansible.cfg file.

Role Variables
--------------

### aspects_iptables_enabled
Boolean
Run task or not. Default is not.

### aspects_iptables_packages:

    aspects_iptables_packages:
      iptablespersistent:
        state: "present"
        Debian: "iptables-persistent"
      firewalld:
        state: "absent"
        RedHat: "firewalld"

Install iptables-persistent on Ubuntu/Debian. Remove firewalld on CentOS.

### aspects_iptables_disable_and_purge
Default: False

When set to `True` along with setting `aspects_iptables_enabled` to `False`, the tasks in [tasks/disableAndPurge.yml](tasks/disableAndPurge.yml) will be run. That will disable the services, and purge the packages from the OS.

It will NOT clear or flush and iptable rules already in place.

Example Playbook
----------------
```yaml
    - hosts: servers
      roles:
         - { role: aspects_iptables, x: 42 }
      vars:
        aspects_iptables_enabled: True
        aspects_iptables_v4_rules:
          00000defaultpolicy:
            enabled: true
            rule: |
              *filter
              :INPUT DROP [0:0]
              :FORWARD DROP [0:0]
              :OUTPUT ACCEPT [0:0]
          1dontdistruptexistingconnections:
            enabled: true
            rule: |
              -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
          2acceptpings:
            enabled: true
            rule: |
              -A INPUT -p icmp -j ACCEPT
          3dontscrewuplo:
            enabled: true
            rule: |
              -A INPUT -i lo -j ACCEPT
          4acceptnewssh:
            enabled: true
            rule: |
              -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
          9acceptmonit:
            enabled: true
            rule: |
              -A INPUT -p tcp -m tcp --dport 2812 -j ACCEPT
          10acceptsnmp:
            enabled: true
            rule: |
              -A INPUT -p tcp -m tcp --dport 161 -j ACCEPT
              -A INPUT -p udp -m udp --dport 161 -j ACCEPT
              -A INPUT -p tcp -m tcp --dport 162 -j ACCEPT
              -A INPUT -p udp -m udp --dport 162 -j ACCEPT
          zzend:
            enabled: True
            rule: |
              -A INPUT -j REJECT --reject-with icmp-host-prohibited
              -A FORWARD -j REJECT --reject-with icmp-host-prohibited
              COMMIT
```

License
-------

MIT
