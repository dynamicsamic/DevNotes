#### Disable `fn` key autouse
```md
Press fn+Esc keys
```
---
#### Connect to MySQL shell
```bash
cmd: mysqlsh
```
---
#### WSL (Windows system for Linux)
```md
# Install WSL
wsl –install # Run as admin via cmd

# Linux machine root
/home/

# Windows root from WSL perspective
/mnt/

# Copy files between machines
[sudo] cp -r /mnt/c/Users/user/dir1/file1 /home/user/dir2/ 

# Run bash inside PowerShell
1. Run wsl
2. Run powershell as admin
3. In powershell prompt enter bash


```
---
#### PowerShell
```bash
# Create new file (not only files, but files first of all)
>>> New-Item -ItemType <File> -Name <filename>
```
---

#### Restrict WSL resource usage
```bash
# Open WSL and navigate to your windows home directory.
cd /mnt/c/Users/<username>

# Check if there is a file named `.wslconfig`.
ls -la | grep '.wslconfig'

# If there is no such file create it.
touch .wslconfig

# Open it and edit
nano .wslconfig

# Insert rows below and save changes. 

# Settings apply across all Linux distros running on WSL 2
[wsl2]

# Limits VM memory to use no more than 4 GB, this can be set as whole numbers using GB or MB
memory=4GB

# Sets the VM to use two virtual processors
processors=2

# Sets amount of swap storage space to 8GB, default is 25% of available RAM
swap=8GB

# Reboot WSL and open again
powershell: wsl --shutdown

# IMPORTANT
# The encoding should be UTF-8 and lines should end with LF (not CR LF)
```
---
#### Get RAM info
```markdown
- Run command prompt as admin
- Type in `wmic memorychip list full` or `wmic memorychip get devicelocator, memorytype, formfactor`
```
---
#### Check specific port usage
```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort <YourPortNumberHere>).OwningProcess
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    519     302   600300      63236              5308   0 mysqld
```
---
#### Check Windows product (activation) key
```cmd
wmic path softwarelicensingservice get OA3xOriginalProductKey
>>> OA3xOriginalProductKey
		AJXXX-XXXXX-XXXXX-XXXXX-XXXXX
```
---
