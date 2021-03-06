# Automation: 

### This project uses Vagrant to create 3 virtualbox ubuntu web servers and configure a loadbalancer on another virtualbox machine. Ansible is used for Provisioning the 4 virtualboxes.

The project was developed and executed on a Ubuntu 16.04 Host with Virtualbox, Vagrant and Ansible packages installed. To install this packages on Ubuntu simply type:

    $ sudo apt-get install virtualbox vagrant ansible

and it will install all other dependecies required by them. The installation of the packages on other operating systems is out of the scope of this project.

In this project, I focused on making dynamic inventory, N machines, reusable tasks, config templates, maximum usage of Ansible facts, separation of roles.

**Vagrantfile**: The Vagrant script creates 3 virtual ubuntu boxes to be used as web servers and 1 extra ubuntu box to be used as load balancer. Load balancer IP: 10.0.15.11, Webservers IP addresses from 10.0.15.21, 22, 23.

**Ansible**: The inventory along with right group is dynamically generated. I have also supplied ansible.cfg file which links to generated inventory and useful to run ansible locally. Used roles to have it organized and to have reusable pieces of code.

**Ansible roles**:

    common - holds tasks and handlers which is common to both load balancer and web server. It installs ufw firewall, nginx and git, and configures the firewall on each virtualbox to allow ssh on port 22 and http on port 80, any other traffic is dropped.
    lb - task and handlers for loadbalancers. It also has config template for ngnix loadbalancer setup (roundrobin).
    web - task and handlers for web servers. It also has sample html and config template for nginx webserver setup.

Ansible playbook pb_web provisions the created web servers by installing git, nginx and copy index.html template to webnodes. One can also pull in code from git and same is tested, But that part of task is commented out for now. I have also configured nginx to add a custom response header to include name of hosts.

Ansible playbook pb_lb provisions the load balancer and also updates the config file for load balancer based on _number of web nodes created. It uses facts from ansible to get the ip address of the hosts and add them in load balancer config of nginx.

During production deployment of web sites, I am aware that one can use pre tasks in ansible to bring out one node at a time from load balancer, deploy web application on that node and after deployment, add the node back to loadbalancer. So that one can do rolling deployment, with one node at a time with zero downtime for customers.

**Test the loadbalancer**: To test the load balancer, testlb.rb file is included which sends N requests to loadbalanced host and reads the host name (from response header) and stores the count of different hosts it has received response from.

**How to run**:

    $ git clone https://github.com/tantosoft/Automation.git
    $ cd Automation
    $ vagrant up

This will start the VM, and run the provisioning playbook (on the first VM startup).

After the installation you can open your favorite web browser and point it to http://10.0.15.11. If you reload the same page you'll see the page served from any of the 3 web servers behind the load balancer.

To re-run a playbook on an existing VM, just run:

    $ vagrant provision
    (optional) $ ansible-playbook pb_web.yml

To run ansible manually inside the project directory, I have included ansible.cfg file which links the generated host file.

Usage of ruby script to test load balancer: -u Specify the name of the host, -n Specify the number of requests, -v Display the version, -h Display this help.

    $ ruby testlb.rb -u "10.0.15.11" -n 1000

Please note that order of supplied arguments cannot be changed. It should be host name(-u) followed by number of requests(-n). The result of running the script is the total number of pages served by each of the web servers behind the load balancer: web1 334, web2 333, web3 333.

See LICENSE for credits.
