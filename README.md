Ansible playbook for deployment of
[otindex](https://github.com/OpenTreeOfLife/otindex), the Open Tree of Life
data store indexer.

In progress. Not ready for production. Tested only on Ubuntu16.

# Installation

Install ansible on the client machine. Requires ansible 2.2 or later. Ansible recommends installing with pip on OS X:

    $ sudo pip install ansible

For OS X users, note that (as of November 2016), the brew version of ansible is only 2.1.

# Configuration

**Hosts**: The `hosts` file contains information about the server where
you are installing otindex. Edit the hostname and usernames for the
production and development servers are set to the correct values.

**Group_vars**:

Here, you define variables that are specific for production and development.
Settings in `group_vars` take precedence over default variables in `roles`.

The `all.yml` file contains settings that are generally common across production
and development. You should only have to edit the database password. If this is
an initial setup, the postgres password will be set to what is in this file. If
the database is already running, then edit it to use the existing password.  

Then cp the server-specific files and edit the appropriate config information
(see file comments for details):

    $ cd group_vars
    $ cp example-production.yml production.yml
    $ cp example-development.yml development.yml

**Roles**

The playbook calls separate [roles](http://docs.ansible.com/ansible/playbooks_roles.html#roles)
for 1) setup of OpenTree data (OTT and phylesystem); 2) postgresql
installation / setup; 3) peyotl installation; and finally 4) otindex
installation / setup. See the `roles/*/defaults/main.yml` and
`roles/*/vars/main.yml` for variables specific to each role, noting that
settings in `vars` take precedence over `defaults`.

Note that there you can't simply run the entire playbook in a virtualenv, so
tasks that require the virtualenv have the following directive:

```
environment:
  VENV: "{{ install_dir }}/../venv"
```

# Pre-prepping SSL certification keys

We now assume (late 2022) that this server uses letsencrypt to maintain files.

    SSLCertificateFile    /etc/letsencrypt/live/opentreeoflife.org/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/opentreeoflife.org/privkey.pem

# Run the playbook
You can limit the playbook to run only for specific servers (production vs
development).

For production:

    $ ansible-playbook otindex.yml -i hosts --limit production

For development:

    $ ansible-playbook otindex.yml -i hosts --limit development




# Other aspects of setting up machine from scratch:

If Ubuntu machine, need to create admin user that has passwordless access

To set up passwordless access

    sudo visudo  

edit to:  

    ## Members of the admin group may gain root privileges  
    %admin ALL=(ALL) NOPASSWD:ALL  

And then create an new admin user named admin

    sudo adduser --disabled-password admin --ingroup admin
    sudo su - admin

then as admin:

    mkdir .ssh
    chmod 700 .ssh
    sudo cat /home/ubuntu/.ssh/authorized_keys 

copy contents into:

    vim .ssh/authorized_keys
    
    chmod 600 .ssh/authorized_keys
 

 ### Adding Swap (from BenRi)
```
 $ sudo swapon --show

$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.8G        142M        7.3G         12M        379M        7.4G
Swap:            0B          0B          0B

$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           798M  804K  797M   1% /run
/dev/xvda1      245G  109G  136G  45% /
tmpfs           3.9G  8.0K  3.9G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop3       18M   18M     0 100% /snap/amazon-ssm-agent/1455
/dev/loop1       90M   90M     0 100% /snap/core/7713
/dev/loop0       18M   18M     0 100% /snap/amazon-ssm-agent/1480
/dev/loop2       89M   89M     0 100% /snap/core/7396
tmpfs           798M     0  798M   0% /run/user/1002

$ sudo fallocate -l 16G /mnt/16GB-manually-added.swap

$ ls -lah /mnt/16GB-manually-added.swap 
-rw-r--r-- 1 root root 16G Oct  9 15:45 /mnt/16GB-manually-added.swap

$ sudo chmod 600 /mnt/16GB-manually-added.swap

$ ls -lah /mnt/16GB-manually-added.swap 
-rw------- 1 root root 16G Oct  9 15:45 /mnt/16GB-manually-added.swap

$ sudo mkswap /mnt/16GB-manually-added.swap 
Setting up swapspace version 1, size = 16 GiB (17179865088 bytes)
no label, UUID=ffbda676-075b-4a09-a072-2339dfd2d77b

$ ls /mnt/
16GB-manually-added.swap

$ sudo swapon /mnt/16GB-manually-added.swap

$ sudo swapon --show
NAME                          TYPE SIZE USED PRIO
/mnt/16GB-manually-added.swap file  16G   0B   -2

$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.8G        217M        7.2G         12M        416M        7.3G
Swap:           15G          0B         15G

$ sudo cp /etc/fstab /etc/fstab.bak

$ echo '/mnt/16GB-manually-added.swap none swap sw 0 0' | sudo tee -a /etc/fstab
/mnt/16GB-manually-added.swap none swap sw 0 0

$ cat /etc/fstab
LABEL=cloudimg-rootfs   /    ext4   defaults,discard    0 0
/mnt/16GB-manually-added.swap none swap sw 0 0

$ cat /etc/fstab.bak 
LABEL=cloudimg-rootfs   /    ext4   defaults,discard    0 0

```
