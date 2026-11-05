# VPS Ansible Setup

This repo setup your personal VPS using Ansible with several server hardening applied. This Ansible playbook assumes you are running on Debian-based Linux OS, e.g. Ubuntu. Before running any of the steps below, modify these variables inside ansible.cfg file, .ini files, and playbook.yml file.
1. VPS-USER, your desired VPS username.
2. VPS-INITIAL-USER, your initial VPS user from cloud provider, could be root or anything else.
3. PATH-TO-INITIAL-SSH-PRIVATE-KEY, path to initial VPS user if using private key.
4. PATH-TO-SSH-PRIVATE-KEY, path to desired VPS user private key, ideally different from initial user's one.
5. VPS-PUBLIC-IP-ADDRESS, your VPS public IP address.
6. YOUR-DOMAIN, your personal domain, e.g. mydomain.com.
7. YOUR-EMAIL, your personal email for Cerbot notification.
8. LIST-OF-CLOUDFLARE-IP-RANGES, list of Cloudflare IP ranges. Check here https://www.cloudflare.com/ips-v4.
9. USER-PASSWORD-HASH, hash of your VPS password. Run this in terminal and copy the output.
```bash
openssl passwd -6
```

This also setup basic web server using Nginx behind Cloudflare for DDoS protection, Bot protection, and static files caching to host your personal website. Put your index.html, custom error page, and any static files inside `public/` directory.

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

3. Generate VPS user password and replace `new_user_password_hash` variable in `playbook.yml` with the generated hash.
```bash
openssl passwd -6
```

4. Run initial setup 

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

5. After this first run, you cannot access VPS using the initial user anymore so you need to change the Ansible inventory file. Enter your VPS user password.
```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

6. To check if your traffic route through Cloudflare, run this `curl` command.
```bash
curl -I yourdomain.com
```
If the response headers contains `Server: cloudflare` and `CF-RAY: [some-alphanumeric-string]-[datacenter-code]`, it means your setup is success. 

