# Configuration of check_usb and nrpe in both nagios server and nagios client/remote hosts

## 1. Install `nrpe` in both `nagios server` and `nagios client/remote` hosts

 1. Install Build essentials

`
sudo apt-get update
sudo apt-get install autoconf gcc libc6 libmcrypt-dev make libssl-dev wget -y
`
 2. Download `nrpe` 

`
cd /tmp ; wget https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz ; tar xzf nrpe-4.0.3.tar.gz ; 
cd nrpe-4.0.3
`

 3. configure, make and install the nrpe

`sudo ./configure --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu`
`sudo make all`
`sudo make install-plugin`

 4. Verify the installation
 
 `ls /usr/local/nagios/libexec/check_nrpe`
 
## 2. This is check_usb plug in develped with ChatGPT

 1. create, copy and save the script as shown below path
 
 `sudo nano /usr/local/nagios/libexec/check_usb.sh`

`
#!/bin/bash

DEVICE_ID="0a12:0001"
STATE_FILE="/tmp/usb_state_$DEVICE_ID.txt"


if ! command -v lsusb &> /dev/null; then
    echo "UNKNOWN: lsusb command not found"
    exit 3
fi


CURRENT_STATE=$(lsusb | grep -c "$DEVICE_ID")


if [ ! -f "$STATE_FILE" ]; then
    echo "$CURRENT_STATE" > "$STATE_FILE"
    if [ "$CURRENT_STATE" -eq 1 ]; then
        echo "OK: USB device $DEVICE_ID is currently connected."
        exit 0
    else
        echo "CRITICAL: USB device $DEVICE_ID is not connected."
        exit 2
    fi
fi

PREVIOUS_STATE=$(cat "$STATE_FILE")


if [ "$CURRENT_STATE" -ne "$PREVIOUS_STATE" ]; then
    echo "$CURRENT_STATE" > "$STATE_FILE"
    if [ "$CURRENT_STATE" -eq 1 ]; then
        echo "CRITICAL: USB device $DEVICE_ID has been plugged in."
        exit 2
    else
        echo "CRITICAL: USB device $DEVICE_ID has been unplugged."
        exit 2
    fi
else
    if [ "$CURRENT_STATE" -eq 1 ]; then
        echo "OK: USB device $DEVICE_ID is connected."
        exit 0
    else
        echo "CRITICAL: USB device $DEVICE_ID is not connected."
        exit 2
    fi
fi
`

save the file.

### NOTE: change USB ID 0a12:0001 with your USB ID

 2. set script executable
 
 `sudo chmod +x /usr/local/nagios/libexec/check_usb.sh`
 
 3. set script user and group to nagios
 
 `sudo chown nagios:nagios /usr/local/nagios/libexec/check_usb.sh`


## 3. Follow the below steps in `nagios client/remote` host

 1. Add and updat `/etc/nagios/nrpe.cfg ` as follows
 
 `sudo nano /etc/nagios/nrpe.cfg`
  - add below 	
    `command[check_usb]=/usr/local/nagios/libexec/check_usb.sh`
  - change below fields as below
    `server_address=0.0.0.0
     server_port=5666`
 2. Restart `nagios-nrpe-server` service
 `sudo systemctl restart nagios-nrpe-server`
 
 3. Verify the service running
 `sudo ss -at | grep nrpe` or `sudo netstate -at | grep nrpe`
 `sudo ss -ntpl | grep 5666` or `sudo netstate -ntpl | grep 5666`
  
## 4. Follow the below steps in `nagios server` host

 1. Verify nagios server and client/remote hosts communication
 `telnet remote_host_ip 5666
 /usr/local/nagios/libexec/check_nrpe -H client-r-remote_host_ip -c check_usb`
 
 
 my case `192.168.122.145`
 `telnet 192.168.122.145 5666
 /usr/local/nagios/libexec/check_nrpe -H 192.168.122.145 -c check_usb`
  
 
 2. Define check_usb command in nagios server `commands.cfg` file
 `sudo nano /usr/local/nagios/etc/objects/commands.cfg`
 
 define command {
    command_name    check_nrpe
    command_line    /usr/local/nagios/libexec/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
 }

 3. Defile nagios client or remote host in nagios server
 `sudo nano /usr/local/nagios/etc/servers/hosts.cfg`
 
 `
 define host {
    use                     linux-server
    host_name               remote_host_name
    alias                   Remote Host
    address                 remote_host_ip
    max_check_attempts      2
    check_period            24x7
    notification_interval   30
    notification_period     24x7
    contact_groups          your_contact_group
}

define service {
    use                     generic-service
    host_name               remote_host_name
    service_description     USB Device 0a12:0001
    check_command           check_nrpe!check_usb
    check_interval          1
    retry_interval          1
    max_check_attempts      3
    check_period            24x7
    notification_interval   30
    notification_period     24x7
    notifications_enabled   1
}

`

## NOTE: replace valus of `host_name`, `address` and USB ID 0a12:0001 with your USB ID. My case nagios-cli` is host_name and 192.168.122.145 is my address

 4. Verify nagios server configure file syntax
 
 `sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`
 
 5. Restart the nagios service
 
 `sudo systemctl restart nagios`
