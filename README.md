# Purpose
This repo is used to keep track of my local environment configurations, to make it easier on me when reinstalling. 
Feel free to use the Ansible playbooks as you wish for your own environment setup. 

# Instructions
**Note:** I'm working on a Fedora workstation primarily, but the playbooks can be easily updated for any Linux 
distribution
1. Copy the hosts_example file to the `hosts` file you will use
   * `cp hosts_example hosts`
1. Properly configure your `hosts` file
1. Run the ansible command
   * `ansible-playbook -i hosts site.yml`
