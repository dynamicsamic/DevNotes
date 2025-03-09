```markdown
- Download and install PuTTY on your machine from its official website (using instructions from the internet). Agree to install PuttyGen to generate or convert secure keys.
- After installation open putty.
- In the `session` section enter the host name or IP adress.
- Port should be 22, connection type - `SSH`.
- Inside the `Connection` section click on `Data` subsection and enter  
- Putty uses its own format of secure keys (PuTTY Private Key File (.ppk)), it does not work with popular ssh keys like RSA, ed_25519 or others. And Linux servers work with `ssh` utility  
```
---
#### Copy files with pscp (PuTTY SCP)
```powershell
## If you installed the complete PuTTY package from oficial website, the `pscp` utility tool is already installed on your PC. Otherwise download it separately.

## To execute pscp move to the directory where the PuTTY package installed. Alternitavely you can add pscp to your PATH and use it from any location of your system.
cd 'C:\Program Files\PuTTY\'

## To use pscp via ssh without having to enter password you need to point the program to the SSH key that is setup in PuTTY. The `-i` option is used like in `scp` tool.

# Copy local file to remote server.
.\pscp.exe -i C:\Users\path\to\putty_key.ppk C:\Users\path\to\local_file.txt user@192.168.1.1:/path/to/remote_server/

# Copy remote file to local machine.
.\pscp.exe -i C:\Users\path\to\putty_key.ppk user@192.168.1.1:/path/to/remote_file.txt "C:\Users\path\to\local_directory\."

# In case of success you should see something like this.
file_name.txt | 90 kB |  90.6 kB/s | ETA: 00:00:00 | 100%

# If you use password based authentication you can provide password via `-pw` option.
.\pscp.exe -pw my_password ...

# A much more secure option would be to use the `-pwfile` option and provide path to the file with password.
.\pscp.exe -pwfile \path\to\file_with_creds ...

# For interactive password typing just omit the -i option, pscp will prompt you for password.
```
---
