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

# POST with multiple headers and nested json.
curl \
	-X POST \
	-H "Content-Type: application/json" -H "Authorization: Bearer ..." \
	-d '{"id": 1, "version": 2, "params": {"period": {"date_from": "2024-12-20", "date_to": "2024-12-27"}, "categories": ["food", "drinks"], "method_name": "getSumByDay"}}' \
	http://example.com/products

# PUT
curl -X PUT -H "Content-Type: application/json" \
-d '{"name": "toy", "category": "adults"}' http://example.com/products

# DELETE
curl -X DELETE http://example.com/products/234944

# GET with authorization token
curl -H "Authorization: Bearer dfk4k04881fj" https://example.com/products/2344

# GET with Basic auth (login-password)
curl -H "Authorization: Basic username:password" https://example.com/products/2344
# OR more secure option to avoid exposing credentials in your shell
curl -H "Authorization: Basic $(base64 <<< cat creds.txt)" https://...
curl -H "Authorization: Basic $(cat creds.txt | base64)" https://...
# OR authomatic base64 encoding by curl with --user option
curl --user "$(cat creds.txt)" -v https://...
# OR prefix domain name with `user:password@`
curl http://usrname:password@example.com/products/234944
# At last you can prompt curl to enter password interactively
curl --user username https://..
#\\ enter your password

# Save data into a file
curl --output outputFile.xml https://example.com
```
---
#### `grep` utility
```Bash
# Grep finds lines containing the pattern. It works with files or stdin

# Find pattern in file.
grep "hello" example.txt

# Find pattern in stdin
cat example.txt | grep "hello"
# OR
echo "hello my friend" | grep "my"

# Find patterns with muiltiple possible characters
cat example.txt | grep "wor[km]*"
cat example.txt | grep "wor[a-z]*"

# grep options
grep --color # sets the color of found patterns to red
gerp -c # count the number of matched lines
grep -o # prints only matched strings each on a separate line without the rest of the line 
grep -n # prefixt each line with a line number
grep -i # ignore case

# Grep can be piped with other utilities. Here we count number of words that follow pattern `wor[a-z]` caseless
cat example.txt | grep "wor[a-z]*" -io | wc -w

# To search several pattern add `-E` option and pipe patterns inside of quotes
cat example.txt | grep -E "hell[a-z]*|wor[a-z]*"

# To perform recursive grepping for a directory use the `-r` option.
grep -r '.*-my_pattern[.]tct$' .

# You can combine this with `-i` (case insensitive) and `-n` (show line number) options.
grep -irn '.*-my_pattern[.]tct$' .

# There is also a possibility to include only specific files (or exclude).
grep -irn '.*-my_pattern' --include ".*txt" .
grep -irn '.*-my_pattern' --exclude ".*txt" .
```
---
#### `sed` utility
```Bash
# sed (stream editor) replaces one string pattern with another

# Basic syntax
sed "s/<old>/new/"

# sed allows using multiple patterns separated with semicolon
sed "s/<old>/<new>/; s/<another>/<the_other>/; s/<more>/<less>/"

# By default sed only replaces complete patterns. If a pattern is a part
# of a longer word, it will leave it as is. To change this behavior
# use `g` command after the last slash.
sed "s/<old>/<new>/g"

# Multiple pattern are possible with the `-E` option (for extended regexp)
sed "s/<old>|<very_old>|<super_old>/<new>/"
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
# Fisrt run the agent and set global variables
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
---
#### File permissions
```bash
# First view file permissions
>>> ls /path/to/file.py
... -rwxr-xr-x 1 tr_sup tr_sup 750 Oct 17 14:40 edu/python/file.py
# first `-` dash means - this is a regular file. Also possible:
- `d` (directory)
- `l` (symlink)
- `s` (socket)
- `c` (character device) # don't know what that is
- `p` (named pipe) # don't know what that is
- `b` (block device) # don't know what that is
- `D` (door) # don't know what that is

# After that goes 3 groups each containing 3 chars r, w, x or -, which mean:
- `r` read permission
- `w` write permission
- `x` executable permission
- `-` permission not set

# The groups they divided into:
- `rwx` # the first group represents the owner's persmissions. In this case onwer of the file can read, write and execute.
- `r-x` # the second group represents the owner's group permissions. In this case all members of the group can read and execute the file but not write to it.
- `r-x` # the third group represents others' permissions. In this case all other users can read and execute the file but not write to it.

# Also permission characters can be represented as digit-values.
- `r` = 4
- `w` = 2
- `x` = 1

# Digits allow to fold a group of three characters into single digit, which forms by summation of each character value. In this case `rwx` = 4 + 2 + 1 = 7, `r-x` = 4 + 1 = 5, and rwxr-xr-x = 755

# After permissions goes number of hard links to a file, in thic case it = 1.
# After that goes the owner's name.
# Then the group's name.
# After that we see the file or directory size in bytes, here it equals to 750.
# Then goes the last modification time.
# And the file name concludes the output.

# The `chmod` utility is used to change file permissions.

# Grant executable permissions for everyone
chmod +x file.txt
-rwxrwxr-x

# Revoke executable permission from everyone
chmod -x file.txt
-rw-rw-r--

# Discard previous permissions and set only executable permissions for everyone
chmod =x file.txt
---x--x--x

# Discard previous permissions and set read and write permissions for everyone
chmod =x file.txt
-rw-rw-r--

# You can also prefix the sign with `u`, `g`, `o` and `a` arguments, which stand for `user`, `group`, `others` and `all` respectively. `a` is the default option if you omit this argument.

# Set read permission to everyone and grant write permission only to user
chmod =r file.txt   # -r--r--r--
chmod u+w file.txt  # -rw-r--r--

# Grant write permission to group
chmod g+w file.txt  # -rw-rw-r--

# Set read permission only to others
chmod o=r file.txt  # -rwxrwxr--

# Multiple permission changes for different types of users may be done in one command, separating each user type with comma.
# Here we set read, write and execute permission to user, read and execute permissions to group and read permission to others.
chmod u=rwx,g=rx,o=r file.txt # no spaces strictly
```
---
#### View file properties
```bash
>>> stat /path/to/file.py
#--------------------------------||-----------------------------------------------#
                                 \/
File:   /path/to/file.py
Size:   750             Blocks: 8          IO Block: 4096   regular file
Device: 8,32            Inode: 82144       Links: 1
Access: (0755/-rwxr-xr-x)  Uid: ( 1000/  tr_sup)   Gid: ( 1000/  tr_sup)
Access: 2024-10-17 14:43:09.574020251 +0300
Modify: 2024-10-17 14:40:03.509616783 +0300
Change: 2024-10-17 14:43:05.452232959 +0300
Birth:  2024-10-17 12:49:07.129337385 +0300 

# Less verbose output possible with ls
>>> ls /path/to/file.py
... -rwxr-xr-x 1 tr_sup tr_sup 750 Oct 17 14:40 edu/python/file.py
```
---
#### `Screen` utility to perform long-running commands
```bash
# screen allows you to run commands and scripts in a detached subtermial.
# It spawnes a new terminal subprocess and runs your command until it finishes or  unitl the system failed.

# List all availavle sessions
screen -ls
	
# Create a new session
screen -S session_name

# Detach a this session from your current terminal
Ctrl+a+d

# Reattach the session to your terminal
screen -r session_name

# If a screen terminal connection was lost before deattaching the session use d option to reattach it.
screen -rd session_name

# Kill a session
screen -S session_name -X quit
```
---
#### Get snapshot of current processes with `ps`
```bash
# Show current processes' snapshot in detailed view
ps -aux # or -auxf to add additional formatting
USER   PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root     1  0.0  0.0 167316 11884 ?        Ss   16:28   0:02 /sbin/init splash
root     2  0.0  0.0      0     0 ?        S    16:28   0:00 [kthreadd]
root     3  0.0  0.0      0     0 ?        S    16:28   0:00 [pool_workqueu...]
root     4  0.0  0.0      0     0 ?        I<   16:28   0:00 [kworker/R-rcu_g]
root     5  0.0  0.0      0     0 ?        I<   16:28   0:00 [kworker/R-rcu_p]
root     6  0.0  0.0      0     0 ?        I<   16:28   0:00 [kworker/R-slub_]
root     7  0.0  0.0      0     0 ?        I<   16:28   0:00 [kworker/R-netns]
root     8  0.0  0.0      0     0 ?        I    16:28   0:00 [kworker/0:0-rcu_gp
................................................................................
sammi 3853  0.0  0.1 344108 24324 ?        Ssl  16:28   0:00 /usr/libexec/xdg...
sammi 3880  0.0  0.1 194244 24588 ?        Sl   16:28   0:00 /usr/libexec/ibus...
root  3904  0.0  0.0      0     0 ?        I<   16:28   0:00 [kworker/u37:2-ttm]
root  3975  0.0  0.2 394348 32444 ?        Ssl  16:28   0:01 /usr/libexec/fwup...
sammi 3976  0.0  0.0 163052  6528 ?        Ssl  16:28   0:00 /usr/libexec/gvfsd
sammi 5882 17.4  1.7 5838376 274496 ?      Sl   16:29   9:06 /snap/vlc/
root  6009  0.0  0.0      0     0 ?        I    16:29   0:00 [kworker/7:2-events]
sammi 6070  0.0  0.1 494264 30772 ?        Sl   16:29   0:00 update-notifier

# What the output means
USER - the user who started the process
PID - process ID
%CPU - percentage of CPU (one CPU core) usage
%MEM - percentage of RAM usage
VSZ - maximum virtual memory size that can be used, KB
RSS - real physical memory used, KB
TTY - wheter the process controls the terminal
STAT - the state of the process;
	  S - interrupted, waits for event to complete
	  l - multi-threaded
	  < - high priority (not nice to other processes)
	  I - idle kernel thread 
	  N - low priority (nice to other processes)
	  R - running or runnable
	  + - in the foreground process
START  - time the process started
TIME - cumulative cpu time in seconds
COMMAND - command/file that started the process with all its arguments and flags

# If the output is too long you can combine it with less command and paginate the output
ps -auxf | less
```
---
#### Display amount of free and used RAM
```bash
free -h # `h` option - human readable

               total        used        free      shared  buff/cache   available
Mem:           2.9Gi       857Mi       1.8Gi       3.6Mi       475Mi       2.1Gi
Swap:          8.0Gi          0B       8.0Gi

# You can also use `m` to output in MB, or `g` to output in GB.
# The default is in KB.
```
---
#### Display file's line endings
```bash
# Use `cat` utility to check line endings 
cat -e file.txt
# For UNIX style LF endings there would be `$` signs.
# For MS style CR LF endings there would be `^M$`

# Use `file` utility to get file metadata
file file.txt
# file.txt: ASCII text => this means the file in unix-style
# file.txt: ASCII text, with CRLF line terminators => means MS-style

# You can convert ms to unix with `dos2unix` utility and vice versa with unix2dos.
# Need to apt install them.
```
---
#### Check message from killed process
```bash
dmesg -T | egrep -i 'killed process'
```
---
#### Install a `.deb` package
```bash
sudo dpkg -i <deb_package_name>
```
---
#### Working with `zip` files
```bash
# Create a zip archive from multiple files
zip my_new_archive_name.zip file1.txt file2.py

# Craete a zip archive from a directory
zip -r my_new_archive_name.zip my_dir

# Update files in an existing archive from a directory
zip -ru existing_archive.zip my_dir

# Set the compression level with integer flags
zip -ru1 existing_archive.zip my_dir # 1 - fast compression
zip -ru9 existing_archive.zip my_dir # 9 - optimal compression

# Show contents of a zip archive without extracting it
unzip -l my_archive.zip
unzip -v my_archive.zip # a more verbose option

# Unzip files in current directory
unzip my_archive.zip

# Unzip files in a particular directory
unzip my_archive.zip /path/to/directory

# If you don't have the rights to access this directory
sudo unzip my_archive.zip /path/to/directory

# Do not print file names during unzipping (quiet mode)
unzip -q my_archive.zip

# Unzip a zip archive protected with password
unzip -P my_password my_archive.zip

# Exclude files from being extracted from a zip archive
unzip my_archive.zip -x file1.txt file2.py

# Exclude a directory
unzip my_archive.zip -x "*.git/*"

# Skip files extraction if such files already exist in a directory
unzip -n my_archive.zip

# Force overwrite existing files with extracted ones
unzip -o my_archive.zip

# Extract only those existing files whose modification time is greater than existing ones
unzip -f my_arhive.zip

# Extract all files and update existing files only if extracting files have greater modification time
unzip -u my_archive.zip
```
---
#### Delete directories with certain names and their contents recursively
```bash
# This will find and delete all directories named `__pycache__` starting from current directory.
# the `-type d` ensures that only directories will be deleted
rm -rf `find . -type d -name __pycache__` 
```
---
#### `tar` utility for working with `tar`, `bzip` and `gzip` archives
```bash
# Tar is capable of creating and extracting files from archives.
# Since tar works with files, directories, links and even devices the first argument that it expects is the file (object) type. Without this argument even the autocomplete would not work.

# List files in a .tar archive.
# f - stands for file, it is used almost every time.
tar tf my_archive.tar
# verbose output (add v letter)
tar tvf my_archive.tar

# This commands also works with other archive types. 
tar tvf my_archive.gz
tar tvf my_archive.tar.gz
tar tvf my_archive.bz2
tar tvf my_archive.tar.bz2

# Create a new archive from a file
# c - create an archive
tar cvf new_archive.tar my_file.txt

# Create a new archive from contents of a directory.
# Recursive operation.
tar cvf new_archive.tar my_dir/

# To create a gz or tgz archive use `z` option
tar czvf new_archive.tar.gz my_dir/
tar czvf new_archive.gz my_dir/
tar czvf new_archive.tgz my_dir/

# To create a bz2 or tbz or tb2 archive use `j` option
tar cjvf new_archive.tar.bz2 my_dir/
tar cjvf new_archive.tar.tbz my_dir/
tar cjvf new_archive.tar.tb2 my_dir/

# Extract files from an archive in current directory.
# The command is same for all types of archives. 
tar xvf my_archive.tar
tar xvf my_archive.tar.bz2
tar xvf my_archive.tbz
tar xvf my_archive.gz
tar xvf my_archive.tgz

# Specify a path to exctract to. Path should exist before extraction.
tar xvf my_archive.tar -C path/to/dir

# Extract a single file from archive
# First list all files in an archive.
# It will output all the files with their paths.
tar tvf my_arhive.tar
# Now copy the output path for your file and extract provide to extract option.
tar xvf my_arhive.tar my_dir/hello.py # Suppose, that is the path provided by the previous command. All directories and files will be created automatically. 

# Extract files that have certain pattern in their names.
# If no files matched command returns with error.
tar xvf my_archive.tar --wildcards '*.txt'
tar zxvf my_archive.tar.gz --wildcards '*.txt'
tar jxvf my_archive.tar.bz --wildcards '*.txt'

# Add a file or directory to archive. Only works with .tar files.
tar rvf existing_archive.tar my_file.txt
tar rvf existing_archive.tar path/to/dir

```