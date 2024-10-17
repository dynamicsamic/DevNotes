---
Created: 2024-07-17T08:26
---
#### Check current ports
```bash
# Check all occupied ports.
sudo lsof -i -P -n
--------------------------------------PRINTS------------------------------------
COMMAND     PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r   792 systemd-resolve   13u  IPv4   9840      0t0  UDP 127.0.0.53:53 
systemd-r   792 systemd-resolve   14u  IPv4   9841      0t0  TCP 127.0.0.53:53 (LISTEN)
...

# Check specific port.
sudo lsof -i:22
--------------------------------------PRINTS------------------------------------
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 27711 root    4u  IPv4 132831      0t0  TCP *:postgresql (LISTEN)
docker-pr 27723 root    4u  IPv6 151769      0t0  TCP *:postgresql (LISTEN)
...

# Check active internet connections.
sudo netstat -tulpn | grep LISTEN
--------------------------------------PRINTS------------------------------------
Active Internet connections (only servers)
Proto Recv-Q Send-Q   Local Address  ForeignAddr  State    PID/Program name 
tcp        0      0   0.0.0.0:5672   0.0.0.0:*    LISTEN   2219/docker-proxy
tcp        0      0   0.0.0.0:5432   0.0.0.0:*    LISTEN   27711/docker-proxy  
tcp        0      0   127.0.0.53:53  0.0.0.0:*    LISTEN   792/systemd-resolve 
tcp        0      0   127.0.0.1:631  0.0.0.0:*    LISTEN   1052/cupsd        
...
```
---
#### System navigation
```bash
cd ~  # Move to $HOME directory.
cd -  # Move to previously active directory.
cd ../  # Move one level up.
```
---
#### Archives
```bash
# Unpack `archive.tar` to /temp.
tar -xvf archive.tar -C /temp
```
---
#### Curl
```bash
# GET; -v show response and request headers; -i for response headers.
curl -v http://example.com/products

# GET; multiple query args needs to be included in quotes.
curl "http://example.com/products/?limit=10&offset=10&category=kids"

# GET; pretty print json response.
curl http://example.com | json_pp

# POST; -H for headers; -d for data; -X for http method.
curl -X POST -H "Content-Type: application/json" \
-d '{"name": "toy", "category": "kids"}' http://example.com/products

# PUT
curl -X PUT -H "Content-Type: application/json" \
-d '{"name": "toy", "category": "adults"}' http://example.com/products

# DELETE
curl -X DELETE http://example.com/products/234944
```
---
#### Generate `rsa` keys for `ssh` connection
```bash
# The `ssh-keygen` is used to generate public and private keys.
# By default this keys are stored in $HOME/.ssh/ directory 
# and has generic names like `rsa...`. To prevent keys mixing up with each
# other it is possible to store this keys in subdirectories as well as give 
# them discriptive names.

mkdir $HOME/.ssh/<new_dir>
ssh-keygen -t rsa -f $HOME/.ssh/<new_dir>/<key_name>
# This script will generate a public and a private keys with provided names
# in provided directory. The public key has `.pub` extension.
# -t option specifies the alogrithm; 
# -f option specifies the file new or existing.

# By default the rsa public key ends with the author information in form of
# <user>@<host>. To change this you can provide the -c option.
ssh-keygen -t rsa -f $HOME/.ssh/<new_dir>/<key_name> -C <CoolJoe>
```
---
#### `Connect` to remote server via `ssh`
```bash
# to connect to remote server use the `ssh` utility.
# Provide the key via `-i <path_to_key>` option and 
# specify the remote server in form `<login>@<public_ip>`
ssh -i $HOME/.ssh/aws/user14 sentry@215.112.101.000
```
---
#### `Copy` files from local machine to remote server via `ssh`
```bash
# To copy files via ssh use the `scp` secure copy utility
# Like with ssh utility you need to specify the key location and
# the remote server address. Additionally you need to specify both paths
# to local the file and the remote files.

scp -i $HOME/.ssh/aws/roger $HOME/docs/file.txt user@111.111.111.111:/home/file.txt
```
---
#### Add keys to ssh-agent
```bash
# Fisrt run the agent
eval $(ssh-agent -s) # => pid 1248

# Add private key
ssh-add mykey

# List all added keys
ssh-add -l

# Delete all added keys
ssh-add -D
```
---
#### Modify `$PATH` variable
```bash
export PATH=$PATH:/my/custom/path
```
---
#### Get information about domain name
```bash
nslookup example.com
			||
			\/
Server:		127.0.0.53     # dns-server name that returned the ip address
Address:	127.0.0.53#53  # ip adress of the dns-server

Non-authoritative answer:
Name:	example.com
Address: 93.184.215.14     # ip-v4 address of example.com
Name:	example.com
Address: 2606:2800:21f:cb07:6820:80da:af6b:8b2c # ip-v6 address of example.com
```
---
#### Install and run  `nginx` on Ubuntu
```bash
# First install ngnix
sudo apt update && sudo apt upgrade -y
sudo apt install ngnix

# Ubuntu is configured to automatically add nginx as service.
# Check if nginx is running as service
sudo service nginx status
# You should see
... Active: active (running) since Wed 2024-10-09 12:51:49 MSK; 13min ago ...

# If you see
... Active: failed (Result: exit-code) since Wed 2024-10-09 12:40:39 MSK; 7min>
# It means Ubuntu could not start nginx service.
# One possible reason is the default 80 port is busy.

# Check 80 port
sudo lsof -i:80
# Example output
COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
apache2 24580     root    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24582 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24583 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24584 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24585 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24586 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
# Here we see that apache2 listen on 80's port

# Check apache2 service status
sudo service apache2 status
>>>  apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
     Active: active (running) since Wed 2024-10-09 12:38:38 MSK; 8min ago ...

# Stop it to free up the port
sudo service apache2 stop

# Start nginx service
sudo service nginx start
```