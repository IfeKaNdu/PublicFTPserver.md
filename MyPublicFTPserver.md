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
### 3. Securing the ftp server  
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

-------------------------------------------------------------- 
### 4. Configuring the vsftpd for the internet  

You can lockdown the server by limiting it to only allow downloads and only from annonymous users inorder to safely share files from the FTP server over the internet.
Back up your current `/etc/vsftpd/vsftpd.conf` file 

```bash 
$ sudo cp -ra /etc/vsftpd/vsftpd.conf   Documents/back_up_/vsftp_backup.conf
```

After the backup, overwrite  `/etc/vsftpd/vsftpd.conf` file with ` /usr/share/doc/vsftpd-*/EXAMPLE/INTERNET_SITE/vsftpd.conf`
```bash
sudo cp -ra  /usr/share/doc/vsftpd-3.0.2/EXAMPLE/INTERNET_SITE/vsftpd.conf   /etc/vsftpd/vsftpd.conf
```
Depending on what your preference is, you can edit the configuration file further if it soothes your preference but mine was left as the above.

-------------------------------------------------------------- 
### 5. Configuring CentOS as a router

Provided you have more than one network interface card (NIC) on a Linux computer, you can configure it to serve as a virtual router.All that is needed is a change to one kernel parameter `ip_forward` that allows packet forwarding.
For immediate and temporal turning on of the ip_forward parameter , type the following commands as root:
```bash
# echo 1 > /proc/sys/net/ipv4/ip_forward
# cat /proc/sys/net/ipv4/ip_forward
 1
```
Inorder to make this change permanent upon system reboot, you must add that value to the /etc/sysctl.conf file, so that it looks like:
`net.ipv4.ip_forward = 1`
After tuning the Linux system to be used as a router, it can also serve the purpose of a firewall between a private network and a public network like the internet.

|  System            |Specification 
| ----------         | ------ |
| Operating System   | CentOS 7 |
| Hostname           | localhost |
| Private Interface  | enp0s25 |
| Public Interface   | wls1 |

The interface connected to our local network is called `private` while the interface connected to the internet or the outer network is called `public` interface.

### Configure the Private interface:

first check the status of the network interface.
```bash
$ nmcli device status
DEVICE      TYPE      STATE        CONNECTION 
virbr0      bridge    connected    virbr0     
enp0s25     ethernet  unavailable  --         
wls1        wifi      unavailable  --         
lo          loopback  unmanaged    --         
virbr0-nic  tun       unmanaged    --   

```
Configure `Private` Interface with necessary settings for the Router setup.
```bash
# nmcli connection add con-name prv0 ifname enp0s25 type ethernet autoconnect yes ip4 192.168.113.10/24 gw4 192.168.113.10
Connection 'prv0' (*) successfully added.
# nmcli connection modify prv0 ipv4.method manual ipv4.dns 192.168.113.10 ipv6.method ignore
# nmcli connection modify prv0 ipv4.never-default yes
# nmcli connection modify prv0 connection.zone internal
# nmcli connection down prv0 ; nmcli connection up prv0
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
```
### Configure Public Interface:
Check status of network devices.
```bash
# nmcli device status
DEVICE      TYPE      STATE        CONNECTION 
virbr0      bridge    connected    virbr0     
enp0s25     ethernet  connected    prv0        
wls1        wifi      unavailable  --         
lo          loopback  unmanaged    --         
virbr0-nic  tun       unmanaged    --   
```
Configure `Public` Interface with necessary settings for the Router setup.
```bash
# nmcli connection add con-name pub0 ifname wls1 type wifi autoconnect yes ssid none ip4 10.176.143.124/16 gw4 10.176.127.1
# nmcli connection modify pub0 ipv4.method auto ipv4.dns 10.176.85.20 ipv6.method ignore
# nmcli connection modify pub0 connection.zone external
# nmcli connection down pub0 ; nmcli connection up pub0
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
```
### Configure Firewall:

```bash
# firewall-cmd --set-default-zone=internal
success
```
Check status of Firewall.
```bash
# firewall-cmd --list-all

internal
  target: default
  icmp-block-inversion: no
  interfaces: enp0s25
  sources: 
  services: ssh mdns samba-client dhcpv6-client
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 


# firewall-cmd --list-all --zone=external

external (active)
  target: default
  icmp-block-inversion: no
  interfaces: wls1
  sources: 
  services: ssh
  ports: 
  protocols: 
  masquerade: yes
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

```
Connect to a client machine client2.localhost in your private network and set the default gateway as follows.
```bash
[client2@localhost ~]# nmcli c a con-name prv0 ifname enp0s25 autoconnect yes type ethernet ip4 192.168.113.11/24 ipv4.dns 192.168.122.10 gw4 192.168.113.10
```
Use the google default dns server ip 8.8.8.8 as an alternative to the dns server 192.168.122.10 ip you already provideded .

-------------------------------------------------------------- 
### 6. Accessing the FTP Server from firefox or with the lftp command
Browsers such as firefox provide an easy interface to connect to the FTP server especially if you simply want to do an anonymous download.For further interactions between the FTP client and the server , using command-line FTP clients is most preferable.

To install lftp command type the following:
```bash
# yum install lftp
```
You will be connected to the FTP server as anonymous user if you use the lftp command with just the name of the FTP server you are trying to access.
By adding the -u username , you can type the user's password when prompted and gain access to the FTP server as the user you logged in as.
Once authenticated, an lftp prompt appears for you to start typing commands hence making connection to the server. Using the get and the put commands, you can use them to download and upload files respectively. Other Linux user commands such as pwd, ls and etc can still run on the lftp prompt. To have the  commands you run interpreted by the client system, you will need to put an exclamation mark(!) infront of the command before running it.
You can use the get and the put command to download and upload files respectively depending on your permission to a file or directory.

**NB** There are also some dedicated graphical FTP clients that can be used such as gFTP, WinSCP,  Free FTP and filezilla.


**Blinks**  
- Annoymous is a special non -authenticated account used by the FTP server. This account can be accessed by anyone because it doesn't require a valid password.
- For the annoymous user , the /var/ftp directory is that user's home directory 
- For regular user, e.g egbuniwe ,when logged in to the server ,my current directory will be /home/egbuniwe , depending on the user's permission , he can change to any part of  the filesystem
- lftp command -oriented FTP client goes into an interative mode after connectin to the server to enable you run commands similar to commands run on the terminal e.g to download or upload files use get and put respecively.
- On firefox, type the Url of the site you want to visit (such as ftp://egbuniwe.com) into the location box if no username or password is added like (ftp://username:MyPasswd5@egbuniwe.com), an annoymous connection is made and the contents of the user's home directory of the site are displayed.Once they are clicked, the files and directories are accessed.
            
            
##### TO BE CONTINUED....
							    
							    
#####                               References 
							
+ Negus, N.  2015: Linux Bible(9th edition). Indianapolis, Indiana: John Wiley & Sons Inc., pp. 347-375,477-497,708-713.

+ 2018, [Ahmer's SysAdmin Recipes ](https://ahmermansoor.blogspot.com/2018/08/setup-linux-machine-as-router.html (accessed July 30,2019)) .


 							


