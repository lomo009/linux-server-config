# linux-server-config

IP 18.217.76.29

## Get Your Server
1. Start new Ubuntu Linux Server instance with Amazon Lightsail
2. Follow instructions to SSH into the server.

## Secure Your Server
### Update all currently installed packages
`sudo apt-get update` to update packages
`sudo apt-get upgrade` to upgrade packages
*select y to continue when prompted
`sudo apt-get install finger`

### Give grader Access
`sudo adduser grader`
create password
edit information
select y to create and save the user

### Grant grader User sudo Permissions
`sudo touch /etc/sudoers.d/grader`
`sudo nano /etc/sudoers.d/grader`
`add grader ALL=(ALL) NOPASSWD:ALL` to the file
exit and save changes

### Configure Key-Based Authentication for grader User
`sudo mkdir /home/grader/.ssh`
`sudo chown grader:grader /home/grader/.ssh`
`sudo chmod 700 /home/grader/.ssh`
`sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/`
`sudo chown grader:grader /home/grader/.ssh/authorized_keys`
`sudo chmod 644 /home/grader/.ssh/authorized_keys`
`sudo nano /home/grader/.ssh/authorized_keys`
Copy details of the udacity_key.rsa.pub file to the authorized_keys file in grader
Exit and save changes.
Can now login with `ssh -i ~/.ssh/udacity_key.rsa grader@18.217.76.29`

### Disable Login for root User
`sudo nano /etc/ssh/sshd_config`
Change `PermitRootLogin prohibit-password` to `PermitRootLogin no`
`sudo service ssh restart`

### Change Timezone to UTC
`sudo timedatectl set-timezone UTC`

### Change Port to 2200
`sudo nano /etc/ssh/sshd_config`
Change `Port 22` to `Port 2200` (4th line from the top)

### Configure UFW Firewall
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sude ufw deny 22`
`sudo ufw allow 2200/tcp`
`sudo ufw allow www`
`sudo ufw allow ntp`
`sudo ufw show added`
`sudo ufw enable`
`sudo ufw status`

## Prepare to Deploy Your Project

## Deploy the Item Catalog Project