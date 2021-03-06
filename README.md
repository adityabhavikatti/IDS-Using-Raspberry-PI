# IDS-Using-Raspberry-PI
Intrusion Detection Using a Raspberry Pi capstone project in Final semester at CSUF.

## Problem Statment
Creating an inexpensive option for homeowners and small time business owners to help level up their network security

## Using Raspberry Pi as an IDS
It is cheap to use and is easy to implement for an enthusiast.

## Tools Used
- Snort- Is an intrusion detection software.
- Rules- Defined to segregate malicious content from harmless content by defining rules.
- Iperf- Software used to help establish the performance of a certain network.
- Sar- Helps to gather computational data of the CPU and the memory load.

## Topology
<img src="/Images/Topology.png?raw=true">

## Method
- With the help of the Iperf software we can create a server and a client and generate network traffic in between the two machines and place a Raspberry Pi as an IDS in between the two machines in the network.

- The remaining tools mentioned will also be installed in the Raspberry Pi

- Once we have the tools running, we can measure the throughput, the CPU and the memory load and finally the analysis of the network using Snort.

# How To Install and Run the Project

## Installing Snort

### Preparing the server

```
sudo apt install -y gcc libpcre3-dev zlib1g-dev libluajit-5.1-dev libpcap-dev openssl libssl-dev libnghttp2-dev libdumbnet-dev bison flex libdnet
```

### Installing from the source

```
mkdir ~/snort_src && cd ~/snort_src
wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
tar -xvzf daq-2.0.6.tar.gz
cd daq-2.0.6
```
### Run the configuration script using its default values, then compile the program with make and finally install DAQ.

```
./configure && make && sudo make install
cd ~/snort_src
```
### Download the source code with wget

```
wget https://www.snort.org/downloads/snort/snort-2.9.15.1.tar.gz

tar -xvzf snort-2.9.15.1.tar.gz
cd snort-2.9.12

./configure --enable-sourcefire && make && sudo make install

```
### Configuring Snort to run in NIDS mode

### Start with updating the shared libraries using the command underneath.

```
sudo ldconfig
sudo ln -s /usr/local/bin/snort /usr/sbin/snort
```

### Setting up username and folder structure

```
To run Snort on Ubuntu safely without root access, you should create a new unprivileged user and a new user group for the daemon to run under.

sudo groupadd snort
sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort
```

### Then create the folder structure to house the Snort configuration, just copy over the commands below.

```
sudo mkdir -p /etc/snort/rules
sudo mkdir /var/log/snort
sudo mkdir /usr/local/lib/snort_dynamicrules
```
### Set the permissions for the new directories accordingly.

```
sudo chmod -R 5775 /etc/snort
sudo chmod -R 5775 /var/log/snort
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
sudo chown -R snort:snort /etc/snort
sudo chown -R snort:snort /var/log/snort
sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules
```
### Create new files for the white and blacklists as well as the local rules.

```
sudo touch /etc/snort/rules/white_list.rules
sudo touch /etc/snort/rules/black_list.rules
sudo touch /etc/snort/rules/local.rules
```
### Then copy the configuration files from the download folder.

```
sudo cp ~/snort_src/snort-2.9.12/etc/*.conf* /etc/snort
sudo cp ~/snort_src/snort-2.9.12/etc/*.map /etc/snort

```
### Using Community rules

```
wget https://www.snort.org/rules/community -O ~/community.tar.gz
```
### Extract the rules and copy them to your configuration folder.
```
sudo tar -xvf ~/community.tar.gz -C ~/
sudo cp ~/community-rules/* /etc/snort/rules
```
### Configuring the network and rule sets

```
sudo nano /etc/snort/snort.conf
```
### Find these sections shown below in the configuration file and change the parameters to reflect the examples here.

```
# Setup the network addresses you are protecting
ipvar HOME_NET 192.168.1.0/24
# Set up the external network addresses. Leave as "any" in most situations
ipvar EXTERNAL_NET !$HOME_NET

# Path to your rules files (this can be a relative path)
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules
# Set the absolute path appropriately
var WHITE_LIST_PATH /etc/snort/rules
var BLACK_LIST_PATH /etc/snort/rules
```
### In the same snort.conf file, scroll down to the section 6 and set the output for unified2 to log under filename of snort.log like below.

```
# unified2
# Recommended for most installs
output unified2: filename snort.log, limit 128
```
### Lastly, scroll down towards the bottom of the file to find the list of included rule sets. You will need to uncomment the local.rules to allow Snort to load any custom rules.

```
include $RULE_PATH/local.rules
If you are using the community rules, add the line underneath to your ruleset as well, for example just below your local.rules line.

include $RULE_PATH/community.rules
```

### Validating settings

```
sudo snort -T -c /etc/snort/snort.conf
```

## Running on your own setup, follow these last few steps:

### Install Sar

```
sudo apt-get install sysstat
```

### Install iperf

```
sudo apt-get install iperf3

```
### In cmd prompt; To Start Iperf server:

```
iperf3 -s
```
### In another cmd prompt

```
ping (RaspberryPi IP Address) -t
```
### Putty1

```
sudo snort -A console -i eth0 -u snort -g snort -c /etc/snort/snort.conf
```
### Putty2

```
iperf3 -c 192.168.1.29 -t 240
```
### Putty3

```
Sar -r 1 120 
Sar -u 1  120 
```
### With the help of the above mentioned tools, we can track the following:

- In Putty window 1 we can start the Snort software and see the network packets in detail and get a detailed report of the network once we have stopped the process, it can list our malicious data packets and alert the admin.
- In Putty window 2 we create the Iperf client and we ping the Iperf server with an amount of time mentioned using the -t flag.
- In Putty window 3 we use the two commands to get the computational data from the CPU and the memory load. 