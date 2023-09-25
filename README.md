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
  - [Step 11: Restarting the server](#step-13-restarting-the-server)
  - [Step 12: Checking](#step-14-checking)
- [Additionally](#additionally)
  - [Installing and configuring Act runner](#installing-and-configuring-act-runner)
  - [Installing and Configuring Ubuntu UFW](#installing-and-configuring-ubuntu-ufw)
  - [Obtaining an SSL certificate](#obtaining-an-ssl-certificate)

## Technologies and software used
| Technologie or software name |     Link      |
|------------------------------|---------------|
| Ubuntu Server 22.04.3        |[Download](https://releases.ubuntu.com/22.04/)|
| Nano                         |[Website](https://www.nano-editor.org/)|
| Git                          |[Website](https://git-scm.com/)|
| Nginx                        |[Website](https://nginx.com/)|
| UFW Ubuntu                   |[Documentation](https://help.ubuntu.com/community/UFW)|
| SSH                          |[wikipedia](https://en.wikipedia.org/wiki/Secure_Shell)|
| PuTTY                        |[Download](https://www.putty.org/)|
| Certbot for Ubuntu           |[Website](https://certbot.eff.org/)|
| Docker                       |[Website](https://www.docker.com/)|
| MySQL                        |[Website](https://www.mysql.com/)|
| Gitea 1.21                   |[Website](https://gitea.com/)|
| Act runner 0.2.5             |[Documentation](https://docs.gitea.com/usage/actions/act-runner)|

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
  ```console 
  clear
  ```
  - To navigate to a directory:  
  ```console
  cd folder
  ```  
  or to navigate from the current folder  
  ```console
  cd folder/folder
  ```  
  or to route from the user's root folder  
  ```console
  cd /folder/folder
  ```  
  or to route from the server root folder  
  ```console
  cd ~/folder/folder
  ```

### Step 1: Connecting to the server
#### First connection
  Let's consider a situation where your hosting provider has given you roor access to the server by login and password.  
  You need to start PuTTY* and connect to the server.  
  *It is assumed that you know how to do this and have read it: [instruction](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html).

  *You should see it:*  
  ```console
  login as:
  ```  
  Type in root  

  ```console
  password:
  ```  
  Type in root password  

  *You should see it:* 
  ```console
  root@your_server_name:~#
  ```
#### After executing [step 3](#step-3-configuring-ssh-access) of this instruction
  WARNING!!! Access to the server will be possible **only by SSH key**.

### Step 2: Upgrade packages
  Commands need to be executed:  
  ```console
  apt update && apt upgrade -y
  ```  
  *This should be done each time before installing packages*

### Step 3: Configuring SSH access
  To work with configuration files, we will need any text editor.  
  *I prefer to use [**Nano**](https://www.nano-editor.org/)*

  **Let's install this text editor, run the command:**  
  ```console
  apt install nano -y
  ```

  **Once *Nano* is installed, we need to configure *OpenSSH*, instruction:**  
  - Run the command  
  ```console
  nano /etc/ssh/sshd_config
  ```

  - In this file you need to find the lines:  
  1. #Port 22 - for security reasons, change the port number, e.g. to 4646 and uncomment the line;
  2. Make sure PermitRootLogin is set to yes (This will give root password access to the server in case something goes wrong [temporary measure]);
  3. PubkeyAuthentication is set to yes and uncommented;
  4. You need to exit and save the changes by pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter".

  **Next, you need to add SSH to autoloader - this should be done with the command:**  
  ```console
  systemctl enable --now ssh
  ```

  **Next, you need to create a directory with access keys by executing the command:**  
  ```console
  mkdir -p ~/.ssh
  ```

  **The next step is to write the public key to a file - this should be done with the command:**  
  ```console
  echo your_public_key >> ~/.ssh/authorized_keys
  ```  
  **Example of a key: ssh-rsa AAAAAkkkkatstyasflRlkqksaJJAUSufisafiIISAFI1gGasfah123/asfasFSAfafsqUUv rsa-key-20230924*

  **Set permissions on the files in the ./ssh directory:**  
  ```console
  chmod -R go= ~/.ssh
  ```

  **Change the owner and group for all files and subdirectories in the ./ssh directory to the system user "root" and its group "root":**  
  ```console
  chown -R root:root ~/.ssh
  ```

  **Restart the SSH service**  
  ```console
  service ssh restart
  ```

  **Try connecting to server via PuTTY using the key**  
  *If everything worked, let's close access by password:*.  
  To do this, follow the familiar steps:  
  - Run the command  
  ```console
  nano /etc/ssh/sshd_config
  ```
  - In this file you need set:
    PermitRootLogin to prohibit-password  
  - Exit and save the changes:  
  Pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter"
  - Restart the SSH service:
  
  ```console
  service ssh restart
  ```

  *Now ALWAYS use only your ssh key to connect to the server*
  **Here we have configured ssh key access to the server and disabled password access**

### Step 4: Install Git
  Now we need to install Git, that's easy:  
  ```console
  apt install git-all -y
  ```

### Step 5: Install Nginx
  The next step is to install Nginx, our actions:  
  1. Install:
  ```console
  apt install nginx -y
  ```
  *We'll configurate it later*

  2. Add Nginx to autoloader:
  ```console
  systemctl enable --now nginx
  ```
  3. To start Nginx after initializing the network connection, replace in the configuration file:  
  ```console
  nano /etc/systemd/system/multi-user.target.wants/nginx.service
  ```
  Change the "After=network.target remote-fs.target nss-lookup.target" line to "After=network-online.target remote-fs.target nss-lookup.target"  

  4. Restart Nginx:  
  ```console
  service nginx restart
  ```  
  5. Status check:  
  ```console
  service nginx status
  ```  
  *I think you can guess where to look. :)*

### Step 6: Install Certbot
  In order for us to be able to get a free SSL certificate for a domain name, we need to install Certbot:  
  1. Install Certbot:  
  ```console
  snap install --classic certbot
  ```  
  2. Check if it's installed Certbot:  
  ```console
  ln -s /snap/bin/certbot /usr/bin/certbot
  ```
  *command executed without error*

### Step 7: Installing Docker and Docker Compose
  1. **Set up Docker's Apt repository** [Detailed information](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)  
  - Step 7.1.1:  
  ```console
  apt install ca-certificates curl gnupg
  ```  
  - Step 7.1.2:  
  ```console
  install -m 0755 -d /etc/apt/keyrings
  ```  
  - Step 7.1.3:  
  ```console
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  ```  
  - Step 7.1.4:  
  ```console
  chmod a+r /etc/apt/keyrings/docker.gpg
  ```  
  - Step 7.1.5:  
  ```console
  echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```  
  - Step 7.1.6:  
  ```console
  apt update -y
  ```  

  2. **Install the Docker packages**  
  ```console
  apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
  ```  

  Add Docker to autoloader:  
  ```console
  systemctl enable --now docker
  ```

  We'll issue launch privileges:
  ```condole
  chmod +x /usr/local/bin/docker-compose
  ```

### Step 8: Installing and Configuring MySQL
  *We will be configuring MySQL to work with Gitea*  

  1. Let's start creating the directory system:  
  ```console
  mkdir /home/mysql && mkdir /home/mysql/giteadb && /home/mysql/giteadb/volume
  ```  
  2. Let's go to the created directory:
  ```console
  cd /home/mysql/giteadb
  ```  
  3. Create a docker-compose.yaml file to create and run MySQL in the docker container:  
  ```console
  nano docker-compose.yaml
  ```
  *Opening in the Nano text editor*  

  4. A script needs to be created. Here is its code:  
  ```yaml
  # WARNING: Use a complex passwords
  version: '3.1'

  networks:
    gitea:
      external: false

  services:
    gitea_db_mysql:
      container_name: gitea_db_mysql      
      image: mysql:8
      ports:
        - "3310:3310"
      restart: always
      networks:
        - gitea
      environment:
        MYSQL_USER: gitea
        MYSQL_PASSWORD: gitea
        MYSQL_ROOT_PASSWORD: root
        MYSQL_ALLOW_EMPTY_PASSWORD: false
      volumes:
        - /home/mysql/giteadb/volume:/var/lib/mysql
      command: ["--character-set-server=utf8mb4", "--collation-server=utf8mb4_unicode_ci"]

    gitea_init:
      image: mysql:8
      depends_on:
        - gitea_db_mysql
      environment:
        MYSQL_ROOT_PASSWORD: root
      command: >
        bash -c "mysql -h gitea_db_mysql -u root -p${MYSQL_ROOT_PASSWORD} -e 'SET old_passwords=0; CREATE DATABASE giteadb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; GRANT ALL PRIVILEGES ON giteadb.* TO gitea; FLUSH PRIVILEGES;'"
  ```  

  - Exit and save the changes:  
  Pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter"  

  5. Let's get our container up and running:  
  ```console
  docker-compose up -d
  ```  
  
  6. Let's verify that the container is running:  
  ```console
  docker ps
  ```  
  *I think you'll know where to look :)*  

  ***We've prepared a database for Gitea***

### Step 9: Installing Gitea
  1. Let's start creating the directory system:  
  ```console
  mkdir /home/applications && mkdir /home/applications/gitea && /home/applications/giteadb/volume
  ```  
  2. Let's go to the created directory:
  ```console
  cd /home/applications/gitea
  ```  
  3. Let's create a new user:  
  ```console
  adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/gitea gitea
  ```  
  *REMEMBER UID and GID*  

  4. Create a docker-compose.yaml file to create and run Gitea in the docker container:  
  ```console
  nano docker-compose.yaml
  ```
  *Opening in the Nano text editor*  

  5. A script needs to be created. Here is its code:  
  ```yaml
  # Note: USER_UID and USER_GID from point 3
  # Note: GITEA__database__USER and GITEA__database__PASSWD from Step 8 point 4
  version: "3.1"

  networks:
    gitea:
      external: false

  services:
    server:
      image: gitea/gitea:1.21
      container_name: gitea
      environment:
        - USER_UID=109
        - USER_GID=113
        - GITEA__database__DB_TYPE=mysql
        - GITEA__database__HOST=gitea_db_mysql:3310
        - GITEA__database__NAME=giteadb
        - GITEA__database__USER=gitea
        - GITEA__database__PASSWD=gitea
      restart: always
      networks:
        - gitea
      volumes:
        - ./home/applications/giteadb/volume:/data
        - /var/gitea/.ssh/:/data/gitea/.ssh
        - /etc/timezone:/etc/timezone:ro
        - /etc/localtime:/etc/localtime:ro
      ports:
        - "3000:3000"
        - "2222:22"
      depends_on:
        - gitea_db_mysql
  ```  

  - Exit and save the changes:  
  Pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter"  

  5. Let's get our container up and running:  
  ```console
  docker-compose up -d
  ```  
  
  6. Let's verify that the container is running:  
  ```console
  docker ps
  ```  
  *I think you'll know where to look :)*  

### Step 10: Configuring Nginx
  1. Now we need to specify in the Nginx settings that the address of our Gitea:
  ```console
  nano /etc/nginx/sites-available/default
  ```
  *Opening in the Nano text editor*  

  2. You need to add this lines:  

  ```bash
  server {
    listen 3000;
    listen [::]:3000;    

    location / {
        client_max_body_size 512M;
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
  ```  

  3. Exit and save the changes:  
  Pressing the keyboard shortcut "Ctrl + x" press "y" and "Enter"  

  4. Restart Nginx:  
  ```console
  service nginx restart
  ```  
  5. Status check:  
  ```console
  service nginx status
  ```  
  *I think you can guess where to look. :)*  

  **You now have the ability to log into your Gitea:**  
  **Open in your browser *http://your_server_ip:3000***  
  *Configure Gitea in the web interface*

### Step 11. Restarting the server
  ```console
  shutdown -r now
  ```  
  *After restarting the server, you can proceed to the next step*

### Step 12. Checking
  **Open in your browser *http://your_server_ip:3000***

## Additionally

### Installing and configuring Act runner
  ***Creating instruction in the process ...***

### Installing and Configuring Ubuntu UFW
  ***Creating instruction in the process ...***

### Obtaining an SSL certificate
  ***Creating instruction in the process ...***