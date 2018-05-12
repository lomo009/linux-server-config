# Udacity Linux Server Config

IP: 13.59.41.139

## Get Your Server
- Start new Ubuntu Linux Server instance with Amazon Lightsail
- Follow instructions to SSH into the server.
- On your local machine log in with:  

`ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem ubuntu@13.59.41.139`  

- Enter `Yes` when prompted

## Update All Installed Packages
- To update packages:  
`sudo apt-get update`  
- To upgrade packages:
`sudo apt-get upgrade`  
- select `Y` to continue when prompted  
`sudo apt-get dist-upgrade`
- select `Y` to continue when prompted  
`sudo apt-get install finger` (optional)  

## Generate Key Pair
`ssh-keygen`
- Save the file to `~/.ssh/udacity_key`

### Give grader Access
- `sudo adduser grader`  
- create password  
- confirm password  
- edit information  
- select `Y` to create and save the user  
`cd /home/grader`
`sudo mkdir .ssh`
`sudo nano .ssh/authorized_keys`
- On local machine run:
`cat ~/.ssh/udacity_key.rsa.pub`
-Copy the contents of udacity_key.rsa.pub to .ssh/auhorized_keys in server terminal
`sudo chown -R grader:grader .ssh`
`sudo chmod -R 700 .ssh`
`sudo chown grader:grader /home/grader/.ssh/authorized_keys`  
`sudo chmod 644 /home/grader/.ssh/authorized_keys`  

### Grant grader User sudo Permissions
`sudo cd /etc/sudoers.d`
`sudo nano grader`
`grader ALL=(ALL) NOPASSWD:ALL`
- exit and save changes  


### Change Timezone to UTC
`sudo timedatectl set-timezone UTC`  