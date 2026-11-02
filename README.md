# VPS Ansible Setup

This repo setup your personal VPS using Ansible with several server hardening applied. Before running any of the steps below, modify several variables in .cfg, .ini, and playbook file. VPS-INITIAL-USER should be the initial VPS username that the cloud provider gives you. VPS-USER then will be your desired VPS username.

This also setup basic web server using Nginx behind Cloudflare for DDoS protection and static files caching.

## Cloudflare setup

In Cloudflare, you need to add your domain so it points to your VPS.

1. Add your domain in Cloudflare Dashboard.
2. Add A and CNAME record in DNS settings.
    1. A record
        Type: A
        Name: @
        IPv4 addres: Your VPS IP address
        Make sure proxy status is enabled
    2. CNAME record
        Type: CNAME
        Name: www
        Target: Your root domain
        Make sure proxy status is enabled
3. Change nameservers in your domain registrar to Cloudflare nameservers.

## Ansible setup

1. Install Ansible inside Python3 virtual environment.
```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
python3 -m pip install ansible
```

2. Remove your VPS IP address from known_hosts file if it ever get added before.
```bash
ssh-keygen -f '/path/to/known_hosts/file' -R 'your_vps_public_ip_address'
```

3. Run initial setup 

If your initial VPS user use password, you need to install `sshpass`.
```bash
sudo apt update
sudo apt install sshpass -y
```

If using private key file.
```bash
ansible-playbook -i setup-with-key.ini playbook.yml
```

If using password. Enter your initial VPS user password.
```bash
ansible-playbook -i setup-with-password.ini playbook.yml --ask-pass
```

4. After this first run, you cannot access VPS using the initial user anymore so you need to change the Ansible inventory file. Enter your VPS user password.
```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

5. To check if your traffic route through Cloudflare, run this `curl` command.
```bash
curl -I yourdomain.com
```
If the response headers contains `Server: cloudflare` and `CF-RAY: [some-alphanumeric-string]-[datacenter-code]`, it means your setup is success. 

