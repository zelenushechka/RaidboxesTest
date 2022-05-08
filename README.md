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
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
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


### Prerequisites

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
  
auto lo  
iface lo inet loopback  
  
auto enp0s3  
iface enpos3 inet static  
    address 10.1.10.0/24  
    netmask 255.255.255.0  
    gateway 10.1.10.132  
    dns-nameservers 8.8.8.8  
  
auto enp0s8  
iface enp0s8 inet dhcp  
* Apply changes.
* Restart.
  ```sh
  sudo /etc/init.d/networking restart
  ```

### Installation

1. Install latest updates of the machine.
* Make sure that OS is up to date.
  ```sh
  sudo apt update
  ```  
  ```sh
  sudo apt upgrade
  ```
2. Install PHP.
  ```sh
  sudo apt install php7.2 php7.2-fpm
  ```  

3. Install NGINX.
   ```sh
   sudo apt install nginx
   ```
4. Set Limits for traffic using the `UFW`
* On UFW.
   ```sh
   sudo ufw enable
   ```
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
5. Setup Wordpress.
* Change directory to HTML.
   ```sh
   cd /var/www/html/
   ```
* Download Wordpress.
   ```sh
   wget https://wordpress.org/wordpress-5.4.10.tar.gz
   ```
   ```sh
   tar -xzvf wordpress-5.4.10.tar.gz
   ```
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
`LUSH PRIVILEGES;`  
`exit`

* Configure NGINX for WordPress.
   ```sh
   mkdir -p /var/www/html/wordpress/public_html
   ```
   ```sh
   cd /etc/nginx/sites-available
   ```
* Create file.

   ```sh
   cat > wordpress.conf << EOF
  server {
              listen 80;
              root /var/www/html/wordpress/public_html;
              index index.php index.html;
              server_name SUBDOMAIN.DOMAIN.TLD;

        access_log /var/log/nginx/SUBDOMAIN.access.log;
            error_log /var/log/nginx/SUBDOMAIN.error.log;

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
  EOF
   ```
   
* Duplicate file to another folder.
   ```sh
   nginx -t
   ```
   ```sh
   cd /etc/nginx/sites-enabled
   ```   
   ```sh
   ln -s ../sites-available/wordpress.conf .
   ```   
* Reload NGINX.   
   ```sh
   systemctl reload nginx
   ```   
6. Switch to nopassword user.
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
7. Switch to nopassword user.   
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


<!-- USAGE EXAMPLES -->
## Usage

You can use this file for solution few task on Ubuntu 18.04. 


<!-- CONTACT -->
## Contact

Anhelina Zelyk - [@zelenushe4ka](https://twitter.com/zelenushe4ka) - [Linkedin](https://linkedin.com/in/zelykanhelina/)- zelikangelina@gmail.com

Project Link: [https://github.com/zelenushechka/RaidboxesTest](https://github.com/zelenushechka/RaidboxesTest)
