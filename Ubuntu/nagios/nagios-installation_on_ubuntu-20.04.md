# Nagios installation in Ubuntu 20.04

## Prerequisites

 - Deploy a fully updated Ubuntu 20.04 Server.
 - Create a non-root user with sudo access.

## 1. Install and Configure Nagios Core

 1. Update the system packages.

    `sudo apt update`
 
 2. Install all the required packages.

    `sudo apt install wget unzip curl openssl build-essential libgd-dev libssl-dev libapache2-mod-php php-gd php apache2 -y`
 
 3. Download Nagios Core Setup files. To download the latest version, visit the official releases site.
 
    `wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz`
 
 4. Extract the downloaded files.

    `sudo tar -zxvf nagios-4.4.6.tar.gz`
 
 5. Navigate to the setup directory.

    `cd nagios-4.4.6`
 
 6. Run the Nagios Core configure script.

    `sudo ./configure`
 
 7. Compile the main program and CGIs.

    `sudo make all`
 
 8. Make and install group and user.

    `sudo make install-groups-users`
 
 9. Add www-data directories user to the nagios group.

    `sudo usermod -a -G nagios www-data`
 
 10. Install Nagios. 
 
      `sudo make install`
 
 11. Initialize all the installation configuration scripts.

      `sudo make install-init`
 
 12. Install and configure permissions on the configs' directory.

      `sudo make install-commandmode`
 
 13. Install sample config files.

      `sudo make install-config`
 
 14. Install apache files.

      `sudo make install-webconf`
 
 15. Enable apache rewrite mode.

      `sudo a2enmod rewrite`
 
 16. Enable CGI config.

      `sudo a2enmod cgi`
 
 17. Restart the Apache service.

      `sudo systemctl restart apache2`
 
 18. Create a user and set the password when prompted.

      `sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users admin`
 
 19. provide access to 'admin' user to monitor the hosts and services

      `sudo vi /usr/local/nagios/etc/cgi.cfg`
 
 NOTE: For bulk replace in vi enter as below `:%s/nagiosadmin/nagiosadmin,admin/gi` and save the file 

## 2. Install Nagios Plugins
 
 1. Download the Nagios Core plugin. To download the latest plugins, visit the plugins download page.

    `cd ~/`

    `wget https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz`

 2. Extract the downloaded plugin.

    `sudo tar -zxvf nagios-plugins-2.3.3.tar.gz`
 
 3. Navigate to the plugins' directory.

    `cd nagios-plugins-2.3.3/`
 
 4. Run the plugin configure script.

    `sudo ./configure --with-nagios-user=nagios --with-nagios-group=nagios`
 
 5. Compile Nagios Core plugins.

    `sudo make`
 
 6. Install the plugins.

    `sudo make install`
   
 7. Add / uncomment below to `nagios.cgi` file.

      `sudo nano /usr/local/nagios/etc/nagios.cfg`

      `cfg_dir=/usr/local/nagios/etc/services`

      Save the file
 
## 3. Verify Nagios Configuration
 
 1. Verify the Nagios Core configuration.

    `sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`
 
 2. Start the Nagios service.

    `sudo systemctl start nagios`
 
 3. Enable Nagios service to run at system startup.

    `sudo systemctl enable nagios`
 
## 4. Access Nagios Web Interface

   Open your web browser and access Nagios web interface via the URL `http://ServerIP/nagios`. For my case:

   `http://192.168.122.162/nagios`

   You have successfully installed Nagios Core on your server. To log in, use admin as your username and the password you set during the user account creation step as your password. You can now access the dashboard and begin configuring Nagios.

## 5. Add Remote Hosts

To monitor hosts, we need to add them to Nagios. By default, Nagios only monitors localhost (the server it's running on). We're going to add hosts that are part of our network to gain even more control. You will need to use the following instructions on all hosts that you want to monitor.

First, install nagios-plugins and nagios-nrpe-server:

`sudo apt-get install nagios-plugins nagios-nrpe-server`

## Configure NRPE

open the /etc/nagios/nrpe.cfg file. Replace the value of `allowed_hosts` with `127.0.0.1,nagios_server_IP` and `server_address` with `0.0.0.0`

Save the file when you are finished.

restart NRPE:

`sudo systemctl restart nagios-nrpe-server` or `service nagios-nrpe-server restart`

## Add the Host to Nagios ( In nagios server )

 1. Create folder and file `/usr/local/nagios/etc/servers/host.cfg` and update file as below

    `sudo mkdir -p /usr/local/nagios/etc/servers/`

    `sudo nano /usr/local/nagios/etc/servers/host.cfg`

      Use the following block as a template. Replace `host` with an appropriate name for your remote host, and update the `host_name`, `alias`, and `address` values accordingly.
      
    ```
      define host {
            use                             linux-server
            host_name                       nagios-cli1
            alias                           My first Apache server
            address                         192.168.122.145
            max_check_attempts              5
            check_period                    24x7
            notification_interval           30
            notification_period             24x7
    }
    ```
    
      save the file
  2. Uncomment following in `/usr/local/nagios/etc/nagios.cfg` file.
  
      Open file

      `sudo nano /usr/local/nagios/etc/nagios.cfg`

      Uncomment below and save the file

      `cfg_file=/usr/local/nagios/etc/services.cfg`

  3. Verify nagios configuration files syntax
  
      `sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`
  
  4. restart or reload the nagios servie
  
      `sudo systemctl restart nagios` or `sudo systemctl reload nagios`

Open your web browser and access Nagios web interface via the URL `http://ServerIP/nagios`. For my case:

   `http://192.168.122.162/nagios`

### To monitor and report a remote host USB, you can refer to the guide on configuring USB checks in Nagios for Ubuntu. You can find it at [USB](/Ubuntu/nagios/nagios-configuration_of_check_usb.md). 

