# Ansible Playbook for Rancher Host Setup and Registration

An Ansible playbook to automatically provison a new host with Docker, 
and the Rancher Agent, then register the new host within Rancher.

## Motivation

If you have a custom host (e.g. a brand new EC2 instance running Ubuntu 14.04),
then you have several required manual steps:

1. SSH into the machine to install Docker  
1. Log in to your Rancher managmeent console then nagivate to -->
Infrastructure --> Hosts --> Add Host --> Custom to copy the docker command
1. Back on the new host, run the docker command, which installs the Rancher
Agent and registers the host with Rancher

That's not overly painful but wastes time if you do it frequently.
More importantly, it's not easy to do this for multiple hosts, as you 
have to do the same steps on each machine.

## How it Works

This playbook will run against your new machine(s) and:

1. Set your machine's hostname according to your inventory (not necessary, just useful)
1. Installs Docker
1. Installs Rancher Agent
1. Queries your Rancher server for correct tokens related to registering the host (this is based on your environment's project ID, more in the Step-By-Step).

## Step-By-Step

### AWS

* Create a new EC2 Instance
    * Ensure ports 22, 4500, and 500 are open
    * Make sure you have the keypair you use for the machine accessible locally
    * Make sure it has a public IP (or you have a way to access the private subnet)
* When it's up, make note of the correct IP address to use

### Local

__Note:__ We recommend you work in a virtual environment.
See [virtualenv](https://virtualenv.readthedocs.org/en/latest/)
and [virtualenvwrapper](https://virtualenvwrapper.readthedocs.org/en/latest/)

__Note 2:__ We strongly recommend encrypting your `group_vars`, `host_vars`, and
`secrets` directories. We use [EncFS](http://www.arg0.net/#!encfs/c1awt)
and check in the encrypted directories to our private repos.

* Make your new virtual environment, make sure to `export PYTHONPATH='pwd'`
* `pip install -r requirements.txt` (installs Ansible)
* Update the credentials required in the `group_vars/rancher-hosts.yaml`
file. Again, don't commit these secrets into a repo.
* Update the `inventories/rancher-hosts` file with your hostname, 
ip address, and path to your keypair.
    * Add as many hosts as you want!
* Retrieve the `project_id` for the Rancher Environment you want this host
to be added to:
    * https://rancher.mydomain.com/v1/projects
    * Look through the list to find the object with the correct `name`, and use the
    associated `project_id` value. i.e. `1a100`
* Run the playbook! Note we add the `project_id` as an `extra-vars` value when running the command.

    ansible-playbook -i ./inventories/rancher-hosts rancher-host-playbook.yaml -s -v --extra-vars "project_id=1a100"

Now just wait a minute or so, then your new host(s) will show up in your Rancher
management console.

### Authors

* Gilman Callsen
* Alejandro Mesa

