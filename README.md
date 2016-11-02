Ansible playbook for deployment of
[otindex](https://github.com/OpenTreeOfLife/otindex), the Open Tree of Life
data store indexer.

In progress. Not ready for production.

# Installation

Requires ansible 2.2 or later on the client. Ansible recommends installing with
pip on OS X:

    $ sudo pip install ansible

Note that (as of November 2016), the brew version of ansible is out of date.

# Configuration

**Hosts**: check the `hosts` file and ensure that the hostname and usernames
for the production and development servers are set to the correct values.

**Group_vars**:

    $ cd group_vars
    $ cp example-production.yml production.yml
    $ cp example-development.yml development.yml

Edit the database connection password in each file. If this is an initial
setup, the postgres password will be set to what is in this file. If the
database is already running, then edit to use the existing password.

# Running the playbook

For production:

    $ ansible-playbook otindex.yml -i hosts --limit production

For development:

    $ ansible-playbook otindex.yml -i hosts --limit development
