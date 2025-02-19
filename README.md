# Self Hosting
I wrote this guide to help me setup and configure VPS machines securely. 

## Where to obtain a VPS
There are many popular services (e.g. [Digital Ocean](https://www.digitalocean.com/), [AWS](https://aws.amazon.com/), etc.) to obtain a VPS. However, as a broke college student, these options can be pretty expensive. Here is a list of much cheaper providers:
- [Hetzner](https://www.hetzner.com/cloud/)
- [Netcup](https://www.netcup.com/en/server/vps)
- [Crunchbits](https://crunchbits.com/vps)
- [Contabo](https://contabo.com/en/vps/)

## Connect to your VPS with SSH
Use the following command to connect to your VPS from your local machine. 
```bash
ssh <username>@<ip-address>
```

## Run updates and upgrades
```bash
sudo apt update
sudo apt upgrade
```
After running the above commands, you may need to restart your VPS. Run the following command, and if the file exists, it means that a restart is required.
```bash
ls /var/run/reboot-required
```
To restart your VPS, use the following command.
```bash
sudo reboot
```

## Change root password
To change the root password, use the following command:
```bash
sudo passwd root
```
Note that ideally, you should disable password-based authentication and use SSH keys instead.

## Create non-root user
Use the following to create a new user and add them to the sudo group.
```bash
sudo adduser <username>
sudo usermod -aG sudo <username>
```
To switch to the new user, use the following command.
```bash
su - <username>
```
You can also try SSHing into the VPS with the new user to verify that it works. From this point on, you should avoid using the root user whenever possible.

## Configure SSH
To improve security, you should disable password-based authentication and use SSH keys instead. This means that you will need to generate an SSH key pair **on your local machine** and add the public key to the VPS. 

First, check out [this guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) for help with generating an SSH key pair.

After generating the SSH key pair, use the following command to add the public key to the VPS. (At this point, you should be logged in as the non-root user.)
```bash
mkdir ~/.ssh
echo "<public-key>" > ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Ensure you test SSH key login with the new user *before* disabling password-based authentication.

## Disable password-based authentication
To disable password-based authentication, open the SSH configuration file using the following command:
```bash
sudo nano /etc/ssh/sshd_config
```
Find the line that says `PasswordAuthentication yes` and change it to `PasswordAuthentication no`. Also, ensure that `PubkeyAuthentication` is set to `yes`. Save the file and restart the SSH service.
```bash
sudo service ssh restart
```
Try SSHing into the VPS (as the non-root user) to verify that password-based authentication has been disabled.

## Disable root login
To disable root login, open the SSH configuration file using the following command:
```bash
sudo nano /etc/ssh/sshd_config
```
Find the line that says `PermitRootLogin ...` and change it to `PermitRootLogin no`. Save the file and restart the SSH service.
```bash
sudo service ssh restart
```

## Set up a basic firewall
Use the following commands to set up a basic firewall using UFW (Uncomplicated Firewall). Only open the ports that are necessary for your VPS. The code below opens ports for OpenSSH, HTTP, and HTTPS.
```bash
sudo apt install ufw
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

## Enable automatic updates
Use the following command to configure automatic updates.
```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Fail2Ban
Fail2Ban is a tool that scans log files and bans IPs that show malicious signs. To install Fail2Ban, use the following command:
```bash
sudo apt install fail2ban
```
Fail2Ban should be automatically started after installation. You can check the status of Fail2Ban using the following command:
```bash
sudo systemctl status fail2ban
```

## Install Docker, Docker Compose, Git, Nginx, etc.
You probably want to install Docker, Docker Compose, Git, and Nginx. The commands for these are not included here, but you can find them in the official documentation for each tool. Many of these should already be installed on your VPS, but if not, you can install them manually.

You may also want to enable memory overcommit to prevent Redis (if used) from crashing. To do this, run the following command:
```bash
echo "vm.overcommit_memory = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Backup your VPS
It's a good idea to set up regular backups for your VPS. There are many ways to do this, including using tools like rsync, Duplicity, or even a cloud-based backup service. Choose a method that works best for you and set it up.
