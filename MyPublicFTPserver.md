### Setting up a Public FTP server on CentOS 7 with virtual router implementation.

Requirements: CentOS 7 on a computer with two or more NICs , firefox web browser, vsftpd package , iftp .

In this article, we will look into the following:

1. The Very Secure FTP Daemon (vsftpd package) installation.
2. Starting the vsftpd service.
3. Securing the FTP server.
4. Configuring the vsftpd for the internet.
5. Configuring CentOS as a router.
6. Accessing the FTP server from the internet.
7. Accessing the FTP server with the lftp command.

-------------------------------------------------------------- 

### 1. The vsftpd FTP server installation

For making files freely availble over networks especially the internet, FTP is mostly used.FTP operates in a client /Server model. A client if authenticated can interatively traverse the filesystem,list files and directories and then download (and sometimes upload) files.

**NB** FTP is insecure because everything sent between the FTP client and server is done in clear text. Inorder words , it's not good for sharing files privately. For private encrypted file sharing,use SSH commands like sftp,scp or rsync.However for open source software repositories , public documents or some openly available data sharing , FTP is still the best choice.

###### Type the following commands as root to install the vsftpd package 
```bash
# yum install vsftpd
```
For general information about the package, run 

```bash
# rpm -qi vsftpd
```

For more on the documentation:
``` bash 
# rpm -qd vsftpd
```

For more on the configuration:

```bash
# man vsftpd.conf
# man vsftpd
```
To view more about the config files type 
 
 ```bash
# rpm -qc vsftpd
 ```
-------------------------------------------------------------- 
### 2. Starting the vsftpd service 
If you just want to use the default settings, no configuration is required to run the vsftpd service.
Before starting the vsftpd service, check whether it is running already.

```bash 
# systemctl status vsftpd.service
 ● vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Tue 2019-07-30 04:00:16 GMT; 5s ago
  Process: 1656 ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf (code=exited, status=0/SUCCESS)
 Main PID: 1668 (code=killed, signal=TERM)

 
```
 The above systemctl showed it as inactive(dead)
 
 ```bash 
# systemctl start vsftpd.service
# systemctl enable vsftpd.service
 ```
```bash 
  systemctl status vsftpd.service
  ● vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2019-07-30 04:04:31 GMT; 12s ago
  Process: 17058 ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf (code=exited, status=0/SUCCESS)
 Main PID: 17061 (vsftpd)
    Tasks: 1
   CGroup: /system.slice/vsftpd.service
           └─17061 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf
```
 Check whether the service is running using the netstat command:
  
```bash
# netstat -tupln | grep vsftpd
 tcp6       0      0 :::21                   :::*                    LISTEN      17061/vsftpd 
```
From the netstat command output, you can see that the vsftpd process (process ID of 17061) is
listening (LISTEN) on all IP addresses for incoming connections on port 21 (0.0.0.0:21) for
the TCP (tcp6) protocol. 
Another quick way to test whether the vsftpd service is actually working is to  put a file in the /var/ftp directory and try to open it from the firefox on the local host
```bash
# echo "Hi From My New FTP Server" > /var/ftp/newFTP.txt
```
From the firefox on the localsystem, type the following into the location box:
 
```bash
ftp://localhost/newFTP.txt
```
if the text *"Hi From my New FTP Server"* appears in the web browser, the vsftpd server is working and accessible from your localsystem.


-------------------------------------------------------------- 
### 3. Securing the ftp server : 
A firwall may prevent your vsftpd FTP srver from fully being accessibele

**Opening up your firewall for FTP**

Firewalls are implemented using iptables rules and managed with the firewalld service.
Firewall rules have traditionally been stored in the /etc/firewalld/zones directory.
Modules are loaded into firewall from the /etc/sysconfig/iptables-config file.
Firewalld service manages firewall rules.

From the GNOME 3 desktop Activities screen, select the firewall icon .Enter the root password when prompted.The firewall configuration window should appear .

Next , to permanently open access to your FTP service, click the configuration box and select permanent . Then add a check box next to ftp under the services tab. This automatically opens TCP port 21(FTP) on your firewall and loads kernel modules needed to allow access to  passive FTP service.

restart the firewall
```bash
# systemctl restart firewalld.service
```
Try again to access the FTPserver from a remote firefox 

**Allowing FTP access in TCP wrappers:**

FTP server can be further secured by adding information to the /etc/hosts.allow and /etc/hosts.deny files to indicate who can or cannot access the selected services.

By default, the hosts.allow and hosts.deny files are empty, which places no restriction on who can access services protected by TCP Wrappers. In the /etc/hosts.allow file, adding lines such as 
```bash
vsftpd: ALL: ALLOW 
```
The line above allows all remote systems to connect to the FTP service vsftpd while changing it to.
```bash
vsftpd:199:171:189 
```
The line above allows a remote client numeric IP address 199:171:189 only, to access the ftpd services.
The /etc/hosts.deny file describes names of the hosts which are not allowed to use the ftpd services 
```bash
ALL:ALL
```
The above line in the /etc/hosts.deny file will deny access to all remote systems that were not allowed access at the /etc/hosts.allow file

**NB** For running an annonymous FTP server that anyone on the internet can access ,use the "ALL" wildcard in the hosts.allow file for the vsftpd service to permit absolutely any host to connect to the FTP service on the Linux system. If you are not running an annoymous FTP site, you might not want to use the "ALL" flag here.

**Configuring SELinux for your FTP Server:**

If SELinux is in Enforcing mode , a few SELinux issues could cause the vsftpd server to not behave as you would like.Check the state of SELinux on your system. 

```bash
# getenforce
Enforcing
```
check the **ftpd_selinux man page**  for information about SELinux settings that can impact the operation of your vsftpd service.

If you cannot access files or directories from your FTP server that you believe should be accessible , you may have to turn off SELinux temporarily to ascertain if the inability to accesss files or directories from your FTP server is from the SELinux.

```bash
# setenforce 0
 ```
When the SELinux is turned into permissive mode and you can access the files and directories , put the system back to Enforcing mode 
```bash
# setenforce 1
```
Then review the ftpd SELinux manpage for the settings that is preventing access.

**NB** 
+ For regular user accounts , the general rule is that if a  user can access a file from the shell, that user can access the same file from an FTP server. So, typically  regular users should at least be able to get(download) and put(upload) files to and from their own homee directories respectively. 
For annoymous user to view or download a file ,at least read permission must be open for other (------r--)

+ Remember to restart the vsftpd service after making any configuration changes. Majority of the configuration is done in the **/etc/vsftpd/vsftpd.conf** file.
User access can be set from vsftpd.conf settings 
```bash 
anonymous_enable=YES
local_enable=YES
 ```
 
 Despite the local_enable setting, SELinux actually prevents vsftpd user from logging in and actually transferring data.Either changing SELinux out of Enforcing mode or setting the correct Boolean allows local accounts to login and transfer data.

Not every user with an account on the linux system has access to the FTP server. The setting *userlist\_enable=YES* in vsftpd.conf mean to deny access to the FTP server for all accounts listed in the /etc/vsftpd/user\_list file.

The /etc/vsftpd/ftpusers file always includes users who are denied access to the server.This settings take precedence to the /etc/vsftpd/user_list file.


**Blinks**  
- Annoymous is a special non -authenticated account used by the FTP server. This account can be accessed by anyone because it doesn't require a valid password.
- For the annoymous user , the /var/ftp directory is that user's home directory 
- For regular user, e.g egbuniwe ,when logged in to the server ,my current directory will be /home/egbuniwe , depending on the user's permission , he can change to any part of  the filesystem
- lftp command -oriented FTP client goes into an interative mode after connectin to the server to enable you run commands similar to commands run on the terminal e.g to download or upload files use get and put respecively.
- On firefox, type the Url of the site you want to visit (such as ftp://egbuniwe.com) into the location box if no username or password is added like (ftp://username:MyPasswd5@egbuniwe.com), an annoymous connection is made and the contents of the user's home directory of the site are displayed.Once they are clicked, the files and directories are accessed.
            
            
##### TO BE CONTINUED....
							    
							    
#####                               References 
							
+ Negus, N.  2015: Linux Bible(9th edition). Indianapolis, Indiana: John Wiley & Sons Inc., pp. 347-375,477-497,708-713.

+ 2018, https://ahmermansoor.blogspot.com/2018/08/setup-linux-machine-as-router.html (accessed July 30,2019).


 							


