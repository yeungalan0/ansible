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

# Raspberry Pi Setup

This repo also supports provisioning a **Raspberry Pi 4** running
Raspberry Pi OS (Debian-based) with Pi-hole (DNS ad-blocker) and
Actual Budget (personal finance tool), both running via Docker Compose.

See the dedicated instructions below.

## Pi Prerequisites

- Raspberry Pi 4 with Raspberry Pi OS (64-bit recommended) installed and on the network.
- Python 3 installed on the Pi (comes pre-installed on Raspberry Pi OS).

## Option A: Run from your workstation over SSH (recommended)

The easiest approach — **Raspberry Pi Imager** can pre-enable SSH and set a
username/password during flashing, so you never need to plug in a monitor:

1. When flashing Raspberry Pi OS with Raspberry Pi Imager, click the **gear icon**
   (or "Edit Settings") and enable:
   - **SSH** (use password authentication)
   - **Set username and password**
   - **Configure WiFi** (if not using Ethernet)
2. Boot the Pi and find its IP address (check your router's DHCP leases, or use
   `ping raspberrypi.local`).
3. **Add your SSH public key(s)** to the repo so they get deployed to the Pi:
   ```bash
   cp ~/.ssh/id_ed25519.pub files/ssh_keys/my_key.pub
   ```
4. **Configure inventory** — copy `hosts_example` to `hosts` (if you haven't already) and
   replace `<PI_IP_ADDRESS>` with your Pi's IP address. Adjust `my_user` if your Pi
   user is not the default `pi`.
5. **Install Ansible Galaxy requirements** (if not done already):
   ```bash
   ansible-galaxy install -r requirements.yml
   ```
6. **Run the Pi playbook** from your workstation:
   ```bash
   ansible-playbook -i hosts playbooks/pi.yml -K
   ```

## Option B: Run locally on the Pi itself

If you don't have SSH access yet (fresh Pi, no monitor-less setup), you can
clone this repo directly onto the Pi and run Ansible locally:

1. On the Pi, install Ansible:
   ```bash
   sudo apt update && sudo apt install -y ansible git
   ```
2. Clone the repo:
   ```bash
   git clone https://github.com/yeungalan0/ansible.git
   cd ansible
   ```
3. **Add your SSH public key(s)** so they get installed into `authorized_keys`:
   ```bash
   cp ~/.ssh/id_ed25519.pub files/ssh_keys/my_key.pub
   ```
   (Or copy a `.pub` file from your workstation onto the Pi via USB drive, etc.)
4. Create your inventory file:
   ```bash
   cp hosts_example hosts
   ```
   Edit `hosts` — in the `[rpi]` section set the host to `localhost ansible_connection=local`
   and set `my_user` to your Pi username.
5. **Install Ansible Galaxy requirements**:
   ```bash
   ansible-galaxy install -r requirements.yml
   ```
6. **Run the Pi playbook locally**:
   ```bash
   ansible-playbook -i hosts playbooks/pi.yml -c local -K
   ```

## Post-Deploy Steps

1. **Set the Pi-hole web UI password** (it is deliberately left blank for security):
   ```bash
   ssh pi@<PI_IP> "docker exec pihole pihole -a -p"
   ```
   You will be prompted to enter a new password.
2. **Point your router's DNS** to the Pi's IP address so all devices on the
   network use Pi-hole for DNS resolution.
3. **Access Actual Budget** at `http://<PI_IP>:5006` and complete the initial setup.
4. **(Optional) Harden SSH** — once you have confirmed key-based SSH access works,
   set `sshd_disable_password_auth: true` in `group_vars/rpi.yml` and re-run
   the playbook to disable password authentication.

## Pi Variables

All Pi-specific variables are in `group_vars/rpi.yml`. Key settings:

| Variable                     | Default            | Description                            |
| ---------------------------- | ------------------ | -------------------------------------- |
| `my_user`                    | `pi`               | User account on the Pi                 |
| `sshd_disable_password_auth` | `false`            | Disable SSH password auth (opt-in)     |
| `pihole_webpassword`         | `""` (empty)       | Pi-hole web password (set post-deploy) |
| `pihole_timezone`            | `America/New_York` | Timezone for Pi-hole                   |
| `pihole_web_port`            | `80`               | Pi-hole web UI port                    |
| `actual_port`                | `5006`             | Actual Budget web port                 |

---

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
