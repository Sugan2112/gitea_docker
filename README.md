# Installing and running Gitea 1.21 in Docker on Ubuntu Server 22.04.3
[https://gitea.com](https://gitea.com/)

## List of contents
- [Technologies and software used](#technologies-and-software-used)
- [Detailed instructions on how to deploy to VPS/VDS server](#detailed-instructions-on-how-to-deploy-to-vps-vds-server)
  - [Step 0: Preparation](#step-0-preparation)
  - [Step 1: Connecting to the server](#step-1-connecting-to-the-server)
  - [Step 2: Upgrade packages](#step-2-upgrade-packages)
  - [Step 3: Configuring SSH access](#step-3-configuring-ssh-access)
  - [Step 4: Install Git](#step-4-install-git)
  - [Step 5: Install Nginx](#step-5-install-nginx)
  - [Step 6: Install Certbot](#step-6-install-certbot)
  - [Step 7: Installing Docker and Docker Compose](#step-7-installing-docker-and-docker-compose)
  - [Step 8: Installing and Configuring MySQL](#step-8-installing-and-configuring-mysql)
  - [Step 9: Installing Gitea](#step-9-installing-gitea)
  - [Step 10: Configuring Nginx](#step-10-configuring-nginx)
  - [Step 11: Installing and configuring Act runner](#step-11-installing-and-configuring-act-runner)
  - [Step 12: Installing and Configuring Ubuntu UFW](#step-12-installing-and-configuring-ubuntu-ufw)
  - [Step 13: Restarting the server](#step-13-restarting-the-server)
  - [Step 14: Checking](#step-14-checking)
- [Additionally](#additionally)
  - [Obtaining an SSL certificate](#obtaining-an-sll-certificate)
  - [Customizing the Gitea template](#customizing-the-gitea-template)
- [Gitea Features](#gitea-features)

## Technologies and software used
- Ubuntu Server 22.04.3 [Download](https://releases.ubuntu.com/22.04/)
- Nano                  [Website](https://www.nano-editor.org/)
- Git                   [Website](https://git-scm.com/)
- Nginx                 [Website](https://nginx.com/)
- UFW Ubuntu            [Documentation](https://help.ubuntu.com/community/UFW)
- SSH                   [wikipedia](https://en.wikipedia.org/wiki/Secure_Shell)
- PuTTY                 [Download](https://www.putty.org/)
- Certbot for Ubuntu    [Website](https://certbot.eff.org/)
- Docker                [Website](https://www.docker.com/)
- MySQL                 [Website](https://www.mysql.com/)
- Gitea 1.21            [Website](https://gitea.com/)
- Act runner 0.2.5      [Documentation](https://docs.gitea.com/usage/actions/act-runner)

## Detailed instructions on how to deploy to VPS/VDS server

### Step 0: Preparation
#### Server rent
  You need to rent a VPS/VDS server and install Ubuntu Server 22.04.3 (preferably the minimalized version).

  Minimum server system requirements:
  - CPU 1x
  - RAM 1024Mb
  - SSD 20Gb
  - Network 100Mb/s

  Usually hosting providers themselves install the operating system and give root access via SSH (We will consider the variant with already installed Ubuntu Server 22.04.3 minimalized + OpenSSH).
#### SSH key pair creation 
  [SSH key pair creation (Documentation)](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html)

#### How to work with the console Ubuntu Server 22.04.3
  - To clear the console:  
  `clear`
  - To navigate to a directory:  
  `cd folder`  
  or to navigate from the current folder  
  `cd folder/folder`  
  or to route from the user's root folder  
  `cd /folder/folder`  
  or to route from the server root folder  
  `cd ~/folder/folder`

### Step 1: Connecting to the server
#### First connection
  Let's consider a situation where your hosting provider has given you roor access to the server by login and password.  
  You need to start PuTTY* and connect to the server.  
  *It is assumed that you know how to do this and have read it: [instruction](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html).

  *You should see it:*  
  `root@your_server_name:~#`
#### After executing [step 3](#step-3-configuring-ssh-access) of this instruction
  WARNING!!! Access to the server will be possible **only by SSH key**.

### Step 2: Upgrade packages
  Commands need to be executed:  
  `apt update && apt upgrade -y`  
  *This should be done each time before installing packages*

### Step 3: Configuring SSH access
  To work with configuration files, we will need any text editor.  
  *I prefer to use [**Nano**](https://www.nano-editor.org/)*

  **Let's install this text editor, run the command:**  
  `apt install nano -y`

  **Once *Nano* is installed, we need to configure *OpenSSH*, instruction:**  
  - Run the command  
  `nano /etc/ssh/sshd_config`

  - In this file you need to find the lines:  
  1. #Port 22 - for security reasons, change the port number, e.g. to 4646 and uncomment the line;
  2. Make sure PermitRootLogin is set to yes (This will give root password access to the server in case something goes wrong [temporary measure]);
  3. PubkeyAuthentication is set to yes and uncommented;
  4. You need to exit and save the changes by pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter".

  **Next, you need to add SSH to autoloader - this should be done with the command:**  
  `systemctl enable --now ssh`

  **Next, you need to create a directory with access keys by executing the command:**  
  `mkdir -p ~/.ssh`.

  **The next step is to write the public key to a file - this should be done with the command:**  
  `echo your_public_key >> ~/.ssh/authorized_keys`  
  **Example of a key: ssh-rsa AAAAAkkkkatstyasflRlkqksaJJAUSufisafiIISAFI1gGasfah123/asfasFSAfafsqUUv rsa-key-20230924*

  **Set permissions on the files in the ./ssh directory:**  
  `chmod -R go= ~/.ssh`

  **Change the owner and group for all files and subdirectories in the ./ssh directory to the system user "root" and its group "root":**  
  `chown -R root:root ~/.ssh`

  **Restart the SSH service**  
  `service ssh restart`

  **Try connecting to server via PuTTY using the key**  
  *If everything worked, let's close access by password:*.  
  To do this, follow the familiar steps:  
  - Run the command  
  `nano /etc/ssh/sshd_config`
  - In this file you need set:  
  PermitRootLogin to prohibit-password
  - Exit and save the changes:  
  Pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter"
  - Restart the SSH service:  
  `service ssh restart`

  *Now ALWAYS use only your ssh key to connect to the server*
  **Here we have configured ssh key access to the server and disabled password access**

### Step 4: Install Git
  Now we need to install Git, that's easy:  
  `apt install git-all -y`

### Step 5: Install Nginx
  The next step is to install Nginx, our actions:  
  1. Install:  
    `apt install nginx -y`  
    *We'll set it up later*
  2. Add Nginx to autoloader:  
    `systemctl enable --now nginx`
  3. To start Nginx after initializing the network connection, replace in the configuration file:  
    `nano /etc/systemd/system/multi-user.target.wants/nginx.service`  
    Change the "After=network.target remote-fs.target nss-lookup.target" line to "After=network-online.target remote-fs.target nss-lookup.target"  
  4. Restart Nginx:  
    `service nginx restart`  
  5. Status check:  
    `service nginx status`  
    *I think you can guess where to look. :)*

### Step 6: Install Certbot
  In order for us to be able to get a free SSL certificate for a domain name, we need to install Certbot:  
  1. Install Certbot:  
    `snap install --classic certbot`  
  2. Check if it's installed Certbot:  
    `ln -s /snap/bin/certbot /usr/bin/certbot`
  *command executed without error*

### Step 7: Installing Docker and Docker Compose
  1. **Set up Docker's Apt repository** [Detailed information](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
    - `apt install ca-certificates curl gnupg`
    - `install -m 0755 -d /etc/apt/keyrings`
    - `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg`
    - `chmod a+r /etc/apt/keyrings/docker.gpg`
    - `echo \ "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \ "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
    - `apt-get update -y`
  2. **Install the Docker packages**
    - `apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y`

### Step 8: Installing and Configuring MySQL

### Step 9: Installing Gitea

### Step 10: Configuring Nginx

### Step 11: Installing and configuring Act runner

### Step 12: Installing and Configuring Ubuntu UFW

### Step 13. Restarting the server

### Step 14. Checking

## Additionally

### Obtaining an SSL certificate

### Customizing the Gitea template

## Gitea Features