# Set SFTP Server
Env: Cent OS 6

## Step
### 1 Add goup, user and permission
    # create a group for SFTP
    [root@dlp ~]# groupadd sftp_users 
    # apply to a user "mysftp" for SFTP only
    [root@dlp ~]# usermod -G sftp_users mysftp
	# usermod bin(require ?)
	[root@dlp ~]# usermod -s /bin/false mysftp
	# chrooted home directory
	[root@dlp ~]# chown root:root mysftp
	# set permissions for home directory
	[root@dlp ~]# chmod 755 mysftp

### 2 Update sshd config file
    [root@dlp ~]# vim /etc/ssh/sshd_config
    # line 132: comment out and add a line like below
    #Subsystem sftp /usr/libexec/openssh/sftp-server
    Subsystem sftp internal-sftp -f AUTH -l VERBOSE
    # add follows to the end
    Match Group sftp_users
      X11Forwarding no
      AllowTcpForwarding no
      ChrootDirectory %h
      ForceCommand internal-sftp
    [root@dlp ~]# /etc/rc.d/init.d/sshd restart 
	# and make sure below
	PasswordAuthentication yes
	UsePAM yes

### 3 Check connection
	[root@localhost ~]# sftp mysftp@10.200.159.83
    Connecting to 10.200.159.83...
    mysftp@10.200.159.83's password: 
    sftp> ls
    downloads  uploads
    sftp> ll
    Invalid command.
    sftp> ls -l
    drwxr-xr-x2 004096 Jul 10 00:22 downloads
    drwxr-xr-x2 004096 Jul 10 00:22 uploads
    sftp> 

### 4 Mind SELinux and chrooted SFTP

## Possible error
> SFTP Error - Couldn't read packet: Connection reset by peer

> Write failed: Broken pipe Couldn't read packet: Connection reset by peer

> sftp Couldn't get handle: Permission denied

Check configuration.

> Network error: Software caused connection abort - FTP - FileZilla
Do NOT chmod to 777/775

> Couldn't create directory: Permission denied
> (Cannot upload file/create dir)
    # setsebool -P ssh_chroot_rw_homedirs on

## Java client
jsch  most used
apache common vfs, less doc and demo, based on jsch  
sshj
