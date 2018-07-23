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

# Pre-prepping SSL certification keys

You will need to get these key files from devapi or another existing server and putting them in these locations on the new machine

    /etc/ssl/private/opentreeoflife.org.key
    /etc/ssl/certs/opentree/STAR_opentreeoflife_org.pem

# Run the playbook
You can limit the playbook to run only for specific servers (production vs
development).

For production:

    $ ansible-playbook otindex.yml -i hosts --limit production

For development:

    $ ansible-playbook otindex.yml -i hosts --limit development
