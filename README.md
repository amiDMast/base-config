# Base-role
## Role consist from the next tasks witch could be run with specific tags:
- **users** (create | delete | edit users)
- **ntp** (Install tzdate, set timezone)
- **ssh** (Install openssh-server, openssh_client, configure sshd)
- **visudo** (Configure /etc/sudoers.d/sudoers file)
- **disks** (Mount attached volume on specify path)
- **firewall**

## Example group_vars for task users:
*For add users*
```yml
adduser:
  users:
    - name: vasya
      pubkey: "{{ lookup('file', '../keys/vasya.pub') }}"
      groups: users
      change: false
    - name: petya
      pubkey: "{{ lookup('file', '../keys/petya.pub') }}"
      groups: sudo
      change: false
```
*For delete users*
```yml
rmuser:
  users:
      - vasya
      - petya
```
*For create|edit user you should use set  **change: true** var in users*
```yml
adduser:
  users:
    - name: vasya
      pubkey: "{{ lookup('file', '../keys/vasya.pub') }}"
      groups: users
      **change: true**
```

## Example group_vars for disks:
```yml
mounts_src:
  - { src: '/dev/sda', path: '/var/lib/logs' }
```

## NTP timezone
*Default timezone* **'Euripe/Kiev'**
*You can define another timezone using variable when you run playbook*
```bash
--tags ntp -e timezone=UTC
```

## Available variables:
```yml
#users
users: users
deployuser: false
adduser_groups: developers

#ntp
timezone: 'Europe/Kiev'

#disks
mount_opts: 'discard,defaults,nofail'
deploy: false 

#ssh
sshd_port: 22
password_authentication: "no"
pubkey_authentication: "yes"

#visudo
BASE_GROUPS:
  - group: 'developers'
    # privileges: '/bin/ln, /bin/su -l deploy, /bin/rm /var/www/admin, /usr/sbin/service admin_uwsgi reload, /usr/sbin/service admin_uwsgi start,$
    privileges: |
      /bin/ln,\
      /bin/su -l deploy

sudoers_files:
  - { src: '../sudoers/sudoers', dest: '/etc/sudoers.d/sudoers' }

# firewall
firewall: true
ipset_list:
  - name: admin_list
    ips:
      - 10.3.60.2
  - name: ipsec_list
    hash: net
    ips:
      - 10.12.112.0/20
  - name: database_list
    hash: net
    ips:
      - 10.3.60.0/23
  - name: consul_list
    hash: net
    ips:
      - 10.3.60.0/23
iptables_list:
  - protocol: icmp
    ipset_list:
      - admin_list
      - ipsec_list
  - protocol: tcp
    tcp: true
    ipset_list:
      - name: admin_list
        dports: 22,7000
      - name: database_list
        dports: 5432,5433,5434,5435,5436
      - name: consul_list
        dports: 8300,8301
  - protocol: udp
    udp: true
    ipset_list:
      - name: ipsec_list
        dports: 500,4500
      - name: consul_list
        dports: 8301
  - protocol: esp
    esp: true
```
