# Ansible Playbooks for Initial Raspberry Pi Lockdown

Simple Ansible playbooks, roles and tasks to lock down and perform initial setup for a new Raspberry Pi.

## Assumptions and Dependencies

These playbooks assume a freshly minted Raspberry Pi running the current version of either Raspbian or Raspbian Lite. Other Raspberry Pi distros exist and [YMMV](https://www.urbandictionary.com/define.php?term=ymmv).

These playbooks also assume that you have [Ansible installed](https://docs.ansible.com/ansible/latest/intro_installation.html) and ready on your control machine.

## Inventory

When a Pi first boots it (usually) receives a DHCP assigned IP address, which the Lockdown playbook changes to a static IP.

To save having to create an inventory file and then immediately update it, these playbooks use a _feature_ of the `--inventory` command line argument for `ansible-playbook` where you can supply an IP address followed _**immediately**_ by a comma so that Ansible know the inventory is a list of hosts (even though there's a single host being targeted).

Like this ... `--inventory 192.168.10.20,`

## Password Playbook

Changes the password for the default `pi` account.

Why the separate playbook? As this playbook changes the password that Ansible is using to authenticate, Ansible will have reload its inventory and host variables, which will fail as the password provided at the start of the playbook is no longer valid.

See [this discussion](https://github.com/ansible/ansible/issues/15227) for more background.

### Usage

```bash
$ ansible-playbook --user pi --ask-pass --inventory 'IP-ADDRESS,' password.yml
```

Running this playbook on a Raspberry Pi with an initial DHCP assigned IP address of `192.168.1.237` will look something like this.

```bash
$ cd plays
$ ansible-playbook --user pi --ask-pass --inventory '192.168.1.237,' password.yml
SSH password:
New pi account password:
confirm New pi account password:

PLAY [Default "pi" account password reset playbook] ****************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.1.237]

TASK [pi-password : Set a new password for the default "pi" account] ***********
changed: [192.168.1.237]

PLAY RECAP *********************************************************************
192.168.1.237              : ok=2    changed=1    unreachable=0    failed=0   
```


## Lockdown Playbook

Performs some initial setup and lockdown on your new Pi.

* Sets the hostname for the Pi
* Creates a new user and deploys an SSH public key for the user
* Disables password authentication and enforces SSH key authentication
* Sets a static IP address, router and DNS servers
* Expands the root filesystem to fill any remaining space on the Pi's SD card

### Usage

```bash
$ cd plays
$ ansible-playbook --user pi --ask-pass --inventory 'IP-ADDRESS,' lockdown.yml
```

Running this playbook on the same Raspberry Pi described above, with a static IP of `192.168.1.2` looks something like this (remember to use the new password for the `pi` account!)

```bash
$ ansible-playbook --user pi --ask-pass --inventory '192.168.1.237,' lockdown.yml
SSH password:
Hostname: dns.vicchi.local
User name: guest
Password:
confirm Password:
Username description: Guest Account
Path to public SSH key: /tmp/id_rsa.pub
Ethernet interface [eth0]:
Static IPv4 address: 192.168.1.2
Routers (comma separated): 192.168.1.1
DNS servers (comma separated) [8.8.8.8,8.8.4.4]:

PLAY [Application server specific playbook] ************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.1.237]

TASK [set-hostname : Set the hostname] *****************************************
changed: [192.168.1.237]

TASK [set-hostname : Update /etc/hosts with new hostname] **********************
changed: [192.168.1.237]

TASK [create-user : Create a (non default) user account] ***********************
changed: [192.168.1.237]

TASK [create-user : Deploy user's SSH key] *************************************
changed: [192.168.1.237]

TASK [disable-passwords : Disable SSH password authentication] *****************
changed: [192.168.1.237]

TASK [static-ip : Configure static IP in  /etc/dhcpcd.conf] ********************
changed: [192.168.1.237] => (item={u'regexp': u'^interface eth[0-9]$', u'line': u'interface eth0'})
changed: [192.168.1.237] => (item={u'regexp': u'^static ip_address', u'line': u'static ip_address=192.168.1.2'})
changed: [192.168.1.237] => (item={u'regexp': u'^static routers', u'line': u'static routers=192.168.1.1'})
changed: [192.168.1.237] => (item={u'regexp': u'^static domain_name_servers', u'line': u'static domain_name_servers=8.8.8.8,8.8.4.4'})

TASK [expand-filesystem : Expand filesystem to fill disk] **********************
changed: [192.168.1.237]

RUNNING HANDLER [static-ip : reboot] *******************************************
changed: [192.168.1.237]

PLAY RECAP *********************************************************************
192.168.1.237              : ok=9    changed=8    unreachable=0    failed=0  
```
