# smbclient.py



## 用途

作为一个smb的客户端可以连接smb服务，访问共享文件，进行相关操作。

相比kali下自带的smbclient，功能更加强大。



## -h 信息

```shell
C:\opt\htb\intranet\impacket\examples> python3 smbclient.py -h
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

usage: smbclient.py [-h] [-file FILE] [-debug] [-hashes LMHASH:NTHASH] [-no-pass] [-k] [-aesKey hex key] [-dc-ip ip address] [-target-ip ip address]
                    [-port [destination port]]
                    target

SMB client implementation.

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>

optional arguments:
  -h, --help            show this help message and exit
  -file FILE            input file with commands to execute in the mini shell
  -debug                Turn DEBUG output ON

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it will
                        use the ones specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)

connection:
  -dc-ip ip address     IP Address of the domain controller. If omitted it will use the domain part (FQDN) specified in the target parameter
  -target-ip ip address
                        IP Address of the target machine. If omitted it will use whatever was specified as target. This is useful when target is the NetBIOS name and you
                        cannot resolve it
  -port [destination port]
                        Destination port to connect to SMB Server
```



## 使用方法

### 小栗子

#### 匿名登录

以 hack the box -> machine -> blue 为例

通过端口扫描知道目标开启了guest用户，且支持匿名登录。

![image-20210415202914966](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210415202914966.png)



#### 密码登录

以 hack the box -> start point -> Pathfinder 为例

之前得到了用户`sandra`的信息如下

```
kerberos :	
	 * Username : sandra
	 * Domain   : MEGACORP.LOCAL
	 * Password : Password1234!
```

![image-20210415203151757](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210415203151757.png)

hash登录同理



### 小功能

help给我们列出了相关的功能

```shell
# help

 open {host,port=445} - opens a SMB connection against the target host/port
 login {domain/username,passwd} - logs into the current SMB connection, no parameters for NULL connection. If no password specified, it'll be prompted
 kerberos_login {domain/username,passwd} - logs into the current SMB connection using Kerberos. If no password specified, it'll be prompted. Use the DNS resolvable domain name
 login_hash {domain/username,lmhash:nthash} - logs into the current SMB connection using the password hashes
 logoff - logs off
 shares - list available shares
 use {sharename} - connect to an specific share
 cd {path} - changes the current directory to {path}
 lcd {path} - changes the current local directory to {path}
 pwd - shows current remote directory
 password - changes the user password, the new password will be prompted for input
 ls {wildcard} - lists all the files in the current directory
 rm {file} - removes the selected file
 mkdir {dirname} - creates the directory under the current path
 rmdir {dirname} - removes the directory under the current path
 put {filename} - uploads the filename into the current path
 get {filename} - downloads the filename from the current path
 mount {target,path} - creates a mount point from {path} to {target} (admin required)
 umount {path} - removes the mount point at {path} without deleting the directory (admin required)
 list_snapshots {path} - lists the vss snapshots for the specified path
 info - returns NetrServerInfo main results
 who - returns the sessions currently connected at the target host (admin required)
 close - closes the current SMB Session
 exit - terminates the server process (and this session)
```

-   shares 查看共享区

    ![image-20210415203408435](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210415203408435.png)

-   use: 打开共享区

-   get/put: 上传/下载文件

-   mkdir/rmdir: 创建/删除文件夹 

-   cd: 切换目录

-   pwd: 当前目录

-   lcd：切换本地（kali）的当前目录

    。。。。。