# Purpose

This repo is used to keep track of my local environment configurations,
to make it easier on me when reinstalling.
Feel free to use the Ansible playbooks as you wish for your own environment setup.

# Instructions

**Note:** I'm working on a Fedora workstation primarily,
but the playbooks can be easily updated for any Linux distribution

1. Install ansible
   - `sudo dnf install ansible`
1. Install ansible galaxy requirements
   - `ansible-galaxy install -r requirements.yml`
1. Copy the hosts_example file to the `hosts` file you will use
   - `cp hosts_example hosts`
1. Properly configure your `hosts` file
1. Install necessary requirements
   - `ansible-galaxy install -r requirements.yml`
1. Run the ansible command
   - `ansible-playbook -i hosts site.yml [-b -k -K -C -c local]`
   - `ansible-playbook -i hosts site.yml -b -K -c local` - Run specifically on localhost
   - `ansible-playbook -i hosts site.yml -b -K -c local --tags dev` - to run a specific role on localhost
     - Note: If using fingerprint auth it may be easier to run `sudo echo` first and then run the above
       command without `-k` or you may experience hanging on gathering facts.

# Troubleshooting

- I was experiencing an issue with the diondonfrost.p10k package (error below). To work around this issue
  I manually added the changes outlined in [this PR](https://github.com/diodonfrost/ansible-role-p10k/pull/3)
  to my local copy of the role (in `https://github.com/diodonfrost/ansible-role-p10k/pull/3`)
  ```
  TASK [diodonfrost.p10k : Enable powerlevel10 instant prompt] *********************************************************************************************************************************************************************************
  fatal: [localhost]: FAILED! => {"changed": false, "msg": "Path /root/.zshrc does not exist !", "rc": 257}
  ```
- If you are hanging on gathering facts most likely there is a sudo issue and you have fingerprint reader
  enabled. In which case, follow the instructions outlined above to enable sudo access and remove the `-K`
  option.

- You can target a specific role to run by using the tags like below:
  ```
  ansible-playbook -i hosts site.yml -b -K -c local --tags always,p10k
  ```
