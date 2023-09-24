# Installing and running Gitea 1.21 in Docker on Ubuntu Server 22.04.3

## List of contents
- [List of technologies used](#list-of-technologies-used)
- [Detailed instructions on how to deploy to VPS/VDS server](#detailed-instructions-on-how-to-deploy-to-vps-vds-server)
  - [Step 0: Preparation](#step-0-preparation)
  - [Step 1: Connecting to the server](#step-1-connecting-to-the-server)
  - [Step 2: Upgrade packages](#step-2-upgrade-packages)
  - [Step 3: Configuring SSH access](#step-3-configuring-ssh-access)
  - [Step 4: Install Nginx](#step-4-install-nginx)
  - [Step 5: Install Certbot](#step-5-install-certbot)
  - [Step 6: Installing Docker](#step-6-installing-docker)
  - [Step 7: Installing and Configuring MySQL](#step-7-installing-and-configuring-mysql)
  - [Step 8: Installing Gitea](#step-8-installing-gitea)
  - [Step 9: Configuring Nginx](#step-9-configuring-nginx)
  - [Step 10: Customizing Gitea](#step-10-customizing-gitea)
  - [Step 11: Installing and configuring Act runner](#step-11-installing-and-configuring-act-runner)
  - [Step 12: Installing and Configuring Ubuntu UFW](#step-12-installing-and-configuring-ubuntu-ufw)
  - [Step 13: Restarting the server](#step-13-restarting-the-server)
  - [Step 14: Checking](#step-14-checking)
- [Additionally](#additionally)
  - [Obtaining an SSL certificate](#obtaining-an-sll-certificate)
## in progress ...
- [Instructions for automatic deployment on a VPS/VDS server](#instructions-for-automatic-deployment-on-a-vps-vds-server)
  - [Step 0: Preparation](#step-0-preparation)
  - [Step 1: Connecting to the server](#step-1-connecting-to-the-server)
  - [Step 2: Upgrade packages](#step-2-upgrade-packages)


## Technologies and software used
- Ubuntu Server 22.04.3 [Download](https://releases.ubuntu.com/22.04/)
- Nano [Website](https://www.nano-editor.org/)
- Nginx [Website](https://nginx.org/)
- UFW Ubuntu [Documentation](https://help.ubuntu.com/community/UFW)
- SSH [wikipedia](https://en.wikipedia.org/wiki/Secure_Shell)
- PuTTY [Download](https://www.putty.org/)
- Certbot for Ubuntu [Website](https://certbot.eff.org/)
- Docker [Website](https://www.docker.com/)
- MySQL [Website](https://www.mysql.com/)
- Gitea 1.21 [Website](https://gitea.com/)
- Act runner 0.2.5 [Documentation](https://docs.gitea.com/usage/actions/act-runner)

## Detailed instructions on how to deposit to VPS/VDS server

### Step 0: Preparation
#### Server rent
  You need to rent a VPS/VDS server and install Ubuntu Server 22.04.3 (preferably the minimalized version).
  Minimum server system requirements:
    - CPU 1x
    - RAM 1024Mb
    - SSD 20Gb
    - Network 100Mb
  Usually hosting providers themselves install the operating system and give root access via SSH (We will consider the variant with already installed Ubuntu Server 22.04.3 minimalized + OpenSSH).
#### SSH key pair creation 
  [SSH key pair creation (Documentation)](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html)

### Step 1: Connecting to the server
#### First connection
  Let's consider a situation where your hosting provider has given you roor access to the server by login and password.
  You need to start PuTTY and connect to the server. *It is assumed that you know how to do this and have read it: [instruction](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html).
  You should see it:
  `root@your_server_name:~#`
#### After executing [step 3](#step-3-configuring-ssh-access) of this instruction
  Access to the server will be possible **only by SSH key**.

### Step 2: Upgrade packages
  Commands need to be executed:  
  `apt update && apt upgrade -y`

### Step 3: Configuring SSH access

### Step 4: Install Nginx

### Step 5: Install Certbot

### Step 6: Installing Docker

### Step 7: Installing and Configuring MySQL

### Step 8: Installing Gitea

### Step 9: Configuring Nginx

### Step 10: Customizing Gitea

### Step 11: Installing and configuring Act runner

### Step 12: Installing and Configuring Ubuntu UFW

### Step 13. Restarting the server

### Step 14. Checking

## Additionally

### Obtaining an SSL certificate
