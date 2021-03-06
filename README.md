# RaidboxesTest


<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/zelenushechka/RaidboxesTest">
    <img src="https://images.raidboxes.io/raidboxes.io/uploads/2021/10/raidboxes-wordmark.svg" alt="Logo" width="80" height="80">
  </a>

<h3 align="center">Raidboxes Test</h3>

  <p align="center">
    Here you can learn how to set up a Wordpress instance from zero and set some specific network permissions.
    <br />
    <a href="https://github.com/zelenushechka/RaidboxesTest"><strong>Explore the docs »</strong></a>
    <br />
    <br />

  </p>
</div>


<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#built-with">Built With</a>
     <li><a href="#getting-started">Getting Started</a></li>
    <li><a href="#installation">Installation</a></li>
     <li><a href="#configure-network-interface">Configure Network Interface</a></li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

<br />
    <img src="https://kinsta.com/de/wp-content/uploads/sites/5/2019/06/wordpress-admin-dashboard.png" alt="Logo" width="400" height="200">
  </a>

### Built With

* [Ubuntu 18.04](https://releases.ubuntu.com/18.04.6/?_ga=2.124596234.1036268425.1651774788-1799098665.1651774788)
* [Nginx](https://www.nginx.com/)
* [PHP](https://www.php.net/)
* [Wordpess](https://wordpress.com/de/)


<!-- GETTING STARTED -->
## Getting Started

To take the test you will need to have:
- VM with Ubuntu 18.04, fresh install
- 2 Network Interfaces(Lower I'll destcibe how to set IP address for your network)

The challenge is as follows:
- Install latest updates of the machine
- Install php with fpm
- Install nginx
- Ensure that traffic coming from outside our network (origin not in 10.1.*) only has
access to content on port 80
- Ensure that port 8088 is only accessible from the internal network
- Setup Wordpress 5.4 (including dependencies) as the only application responding on
port 80 (no need to perform the WP Install)
- Create a user with sudo nopasswd rights like root and its respective keys
- Disallow root access through ssh


### Installation
1. Install latest updates of the machine.
* Make sure that OS is up to date.
  ```sh
  sudo apt update
  ```  
  ```sh
  sudo apt upgrade
  ```
2. Switch to nopassword user.
* On UFW.
   ```sh
   sudo ufw enable
   ```
   ```sh
   sudo -i
   ```
   ```sh
   sudo adduser --shell /bin/bash myuser
   ```   
   ```sh
   sudo usermod -aG sudo myuser
   ```  
   ```sh
   sudo echo "myuser ALL=(ALL) NOPASSWD: ALL" | (EDITOR="tee -a" visudo)
   ```
* Generate SSH, create sudo user with only an SSH key

   ```sh
   sudo ssh-keygen
   ``` 
   ```sh
   sudo apt install openssh-server
   ``` 
   ```sh
   sudo ufw allow ssh
   ``` 
   
   ```sh
   sudo adduser --shell /bin/bash --system --group myuser1
   ``` 
   ```sh
   sudo mkdir /home/myuser1/.ssh
   ```
   ```sh
   sudo cp -Rfv /root/.ssh /home/myuser1/
   ```  
   ```sh
   sudo chown -Rfv myuser1:myuser1 /home/myuser1/.ssh
   ``` 
   ```sh
   sudo chown -R myuser1:myuser1 /home/myuser1
   ``` 
   ```sh
   sudo gpasswd -a myuser1 sudo
   ``` 
   ```sh
   sudo  echo "myuser1 ALL=(ALL) NOPASSWD: ALL" | (EDITOR="tee -a" visudo)
   ``` 
* Disable root SSH login.   
   ```sh
   nano /etc/ssh/sshd_config
   ```    
* Uncomment "PermitRootLogin" and type "no":

#LoginGraceTime 2m  
**PermitRootLogin no**  
#StrictModes yes  
#MaxAuthTries 6  
#MaxSessions 10  

   ```sh
   systemctl restart ssh
   ```    


3. Install PHP.
  ```sh
  sudo apt-get install php7.2 php7.2-cli php7.2-fpm php7.2-mysql php7.2-json php7.2-opcache php7.2-mbstring php7.2-xml php7.2-gd php7.2-curl
  ```  

4. Install NGINX.
   ```sh
   sudo apt install nginx
   ```
5. Setup Wordpress.
* Install My SQL database.
   ```sh
   sudo apt install mysql-server 
   ```
   ```sh
   sudo mysql_secure_installation
   ```
   ```sh
   mysql -u root -p
   ```
* Create database.   
`CREATE DATABASE wordpress_db;`
`GRANT ALL ON wordpress_db.* TO 'wpuser'@'localhost' IDENTIFIED BY 'Passw0rd!' WITH GRANT OPTION;`
`FLUSH PRIVILEGES;`  
`exit`
* Create new directory.
   ```sh
   mkdir -p /var/www/html/wordpress/public_html
   cd /var/www/html/wordpress/public_html
   ```
* Download Wordpress.
   ```sh
   wget https://wordpress.org/wordpress-5.4.10.tar.gz
   ```
   ```sh
   tar -xzvf wordpress-5.4.10.tar.gz
   mv wordpress/* .
   rm -rf wordpress wordpress-5.4.10.tar.gz
   ```
* Change the ownership and apply correct permissions.
   ```sh
   chown -R www-data:www-data *
   chmod -R 755 *
   ```
* Configure NGINX for WordPress.
   ```sh
   cd /etc/nginx/sites-available
   ```
* Change port default server to 8080 in `default` file.
   ```sh
   nano default
   ``` 
   and change here port from 80 to 8080 **(listen 8080;)** in the two places.
* Create new file.

   ```sh
   touch wordpress.conf
   nano wordpress.conf
   ```
* Add content.
   ```sh
  server {
              listen 80;
              root /var/www/html/wordpress/public_html;
              index index.php index.html;
              server_name wpexample.com;

        access_log /var/log/nginx/wpexample.wordpress.access.log;
            error_log /var/log/nginx/wpexample.wordpress.error.log;

              location / {
                           try_files $uri $uri/ =404;
              }

              location ~ \.php$ {
                           include snippets/fastcgi-php.conf;
                           fastcgi_pass unix:/run/php/php7.2-fpm.sock;
              }

              location ~ /\.ht {
                           deny all;
              }

              location = /favicon.ico {
                           log_not_found off;
                           access_log off;
              }

              location = /robots.txt {
                           allow all;
                           log_not_found off;
                           access_log off;
             }

              location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                           expires max;
                           log_not_found off;
             }
  }
   ```
   
* Create a symbolic link for this file.
   ```sh
   cd /etc/nginx/sites-enabled
   ln -s ../sites-available/wordpress.conf .
   ```    
* Reload NGINX.   
   ```sh
   systemctl reload nginx
   ```   
 
6. Set Limits for traffic using the `UFW`
* Get status UFW.
   ```sh
   sudo ufw status
   ```
* Deny all connections.
   ```sh
   sudo ufw default deny outgoing
   ```
   ```sh
   sudo ufw default deny incoming
   ```
* Allow specific ports.
   ```sh
   sudo ufw allow out 80
   ```
   ```sh
   sudo ufw allow 'Nginx HTTP'
   ```
* Status.
   ```sh
   sudo ufw status verbose 
   ```
   
### Configure Network Interface

Below is a list of the things you need to do and how to set them up. Set the IP for the network interface.
* Get a list of available interfaces.
  ```sh
  ifconfig -a
  ```
* Go to file.
  ```sh
  sudo nano /etc/network/interfaces
  ```
* Edit file as mentioned below.  
  
  ```text
  auto lo  
  iface lo inet loopback  

  auto enp0s3  
  iface enp0s3 inet static  
      address 10.1.10.0/24  
      netmask 255.255.255.0  
      gateway 10.1.10.132  
      dns-nameservers 8.8.8.8
  ```  
* Apply changes.
* Restart.
  ```sh
  sudo /etc/init.d/networking restart
  ```

   
<!-- USAGE EXAMPLES -->
## Usage

You can use this file for solution few task on Ubuntu 18.04. 
**P.S. To complete the WordPress installation, go to your localhost: http://127.0.0.1/.**


<!-- CONTACT -->
## Contact

Anhelina Zelyk - [@zelenushe4ka](https://twitter.com/zelenushe4ka) - [Linkedin](https://linkedin.com/in/zelykanhelina/)- zelikangelina@gmail.com

Project Link: [https://github.com/zelenushechka/RaidboxesTest](https://github.com/zelenushechka/RaidboxesTest)
