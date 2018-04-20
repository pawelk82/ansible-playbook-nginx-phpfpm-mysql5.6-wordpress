Ansible Roles: Install Nginx + PHP-FPM + MySQL5.6 + Wordpress on Debian
==================

This roles install latest version of Nginx, PHP-FPM(the version of the package can be specified in the variable `php_ver`), MySQL5.6 (Oracle's APT Repo), Wordpress and WP-CLI and configures Wordpress

Configure host
------------------

Configure the remote host parameters in the file `hosts`

Example:

    [webservers]
    debian.example.com ansible_ssh_host=192.168.56.5

Specify a FQDN name, it is used in variables!

Roles Variables
------------------

All variables for roles are stored in a file:

    config/config.yml

SSH connect to the remote host
------------------

If the keys are generated and copied to the remote host - specify the user name in the variable `ansible_ssh_user`. 
If the remote host is accessed using a password, uncomment the variable `ansible_ssh_pass` and specify the password.

It is understood that the file `/etc/sudores` on the remote host contains a parameter `your_ssh_user_name ALL=(ALL) NOPASSWD: ALL`

Role: Configure
------------------

This role:

* Install Oracle's APT Repo and add MySQL repo key
* Download and install WP-CLI
* Update hosts and hostanme file
* Update all packages to the latest version

Role: Nginx
------------------

This role install latest version of Nginx and disable default site

Role: PHP-FPM
------------------

This role install:

* php-cli 
* php-common 
* php-mysql 
* php-gd 
* php-fpm 
* php-cgi 
* php-mcrypt
* php-pear

The version is specified in the variable `php_ver` in `config/config.yml`

Role: MySQL
------------------

This role install:

* mysql-server
* python-mysqldb

The MySQL root user password is set in the variable `mysql_root_password` in `config/config.yml`

Role: Wordpress
------------------

This role:

* Creating MySQL database (variable `db_name` in `config/config.yml`), creating MySQL database user (variable `db_user` and `db_password` in `config/config.yml`) for Wordpress
* Download WordPress latest version, create directory (variable `wpdirectory` in `config/config.yml`), update WordPress config file and change permission for all Wordpress directory and file
* Update WordPress config file
* Copy and activate Nginx configuration (file: '/roles/wordpress/templates/wordpress.conf')

```
server {
  listen 80 default_server;

  root {{ wpdirectory }}/wordpress;
  index index.php index.html index.htm;

  server_name {{ domain }};

  location / {
    try_files $uri $uri/ /index.php?q=$uri&$args;
  }

  error_page 404 /404.html;

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/www;
  }

  # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
  location ~ \.php$ {
    try_files $uri =404; 
    include /etc/nginx/fastcgi.conf;
    fastcgi_pass unix:/run/php/php{{ php_ver }}-fpm.sock;
  }
}

server {
  listen 80;
  server_name www.{{ domain }};
  location / {
    return 301 http://{{ domain }};
  }
}
```

Install playbook
------------------

Plays roles:

* configure
* nginx
* php-fpm
* mysql
* wordpress

Run the role:

    ansible-playbook -i hosts install.yml

Configure playbook
------------------

This playbook configure wordpress. Set variables in `config/config.yml`: 
```
wordpress_url 
wordpress_title 
wordpress_user 
wordpress_password 
wordpress_user_email
```
Run the role:

    ansible-playbook -i hosts configure.yml

Enter site address in the browser. You should see the main page with the standard WordPress template

# ansible-playbook-nginx-phpfpm-mysql5.6-wordpress
