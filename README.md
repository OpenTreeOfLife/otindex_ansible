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
  VENV: "{{ install_dir }}/venv"
```

# Pre-prepping SSL certs

You will need to get these files from devapi or an existing server and plac ethem on the new machine


/etc/apache2/sites-available/opentree_ssl.conf  -> (I'm working on a template for this...)
/etc/ssl/private/opentreeoflife.org.key



STAR_opentree


Getting the key file requires moving to ~ and changing the owner for scp.

    sudo cp /etc/ssl/private/opentreeoflife.org.key ~/
    sudo chown admin opentreeoflife.org.key




# Half-working

On ot40 currently requires ssh ing in and running

    sudo a2enmod rewrite
    sudo a2enmod proxy



ing the playbook

You can limit the playbook to run only for specific servers (production vs
development).

For production:

    $ ansible-playbook otindex.yml -i hosts --limit production

For development:

    $ ansible-playbook otindex.yml -i hosts --limit development

# After running ansible setup:
ssh in,


    source otindex/venv/bin/activate
    cd /home/admin/otindex/ott
    rm *
    wget http://files.opentreeoflife.org/ott/ott3.0/ott3.0.tgz
    tar -xzvf ott3.0.tgz
    mv ott ott3.0


    mv otindex/.peyotl ~/.peyotl
    check that the paths are correct in the peyptl

## If no database dump:
    cd /home/admin/otindex/otindex/otindex/scripts
    chmod +755 run_setup_scripts.sh
    ./run_setup_scripts.sh ../../development.ini 

