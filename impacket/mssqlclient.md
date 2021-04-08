# mssqlclient



## 用途

用于登录 mssql ，可以使用两种方法：

-   mssql用户名、密码。
-   Windows认证。
    -   Windows hash认证
    -   Windows 密码认证



因此，适合在有数据库账号密码或者windows账号密码的情况下使用。



## -h 信息

```shell
C:\opt\htb\intranet\impacket\examples> mssqlclient.py -h           
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

usage: mssqlclient.py [-h] [-port PORT] [-db DB] [-windows-auth] [-debug] [-file FILE] [-hashes LMHASH:NTHASH] [-no-pass] [-k] [-aesKey hex key]
                      [-dc-ip ip address]
                      target

TDS client implementation (SSL supported).

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>

optional arguments:
  -h, --help            show this help message and exit
  -port PORT            target MSSQL port (default 1433)
  -db DB                MSSQL database instance (default None)
  -windows-auth         whether or not to use Windows Authentication (default False)
  -debug                Turn DEBUG output ON
  -file FILE            input file with commands to execute in the SQL shell

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials
                        cannot be found, it will use the ones specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  -dc-ip ip address     IP Address of the domain controller. If ommited it use the domain part (FQDN) specified in the target parameter
```



## 使用方法

获得用户名和密码的途径有很多，写几个我觉得可以的，遇到过的用红色标出：

-   <font color='red'>网站的配置文件。</font>
-   <font color='red'>备份文件。</font>
-   <font color='red'>smb共享文件。</font>
-   <font color='red'>procdump + mimikatz</font>。
-   键盘监听器（钓鱼）。

#### 小栗子

假设得到了可以登录sql server的用户名和密码，现在需要登录到mssql中去，就可以使用`mssqlclient.py`工具。

以htb -> start_point -> Archetype 靶机作为栗子。

通过smb得到了`prod.dtsConfig`，里面写着mssql的用户名和密码。

```xml
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

看到UserId，这应该是以Windows认证登录mssql的例子。

使用方法为：

```shell
C:\opt\htb\intranet\impacket\examples> python3 mssqlclient.py ARCHETYPE/sql_svc[:password]@10.10.10.27 -windows-auth
```

如果不填写`[:password]`字段的话，交互时会提示你输入，登录成功后，得到一个sql shell。

![image-20210408175314123](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210408175314123-20210408204010491.png)

#### 小功能

它内嵌了一些小功能，使用help命令查看。

![image-20210408175354110](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210408175354110-20210408204011078.png)

-   `lcd (path)`： 切换本地目录（也就是主机kali的目录），在本地执行。

-   `exit`：退出脚本。

-   `enable_xp_cmdshell`：打开`xp_cmdshell`，开启后可以得到os shell，需要sa权限。

    出现这个结果代表以及打开了

    ![image-20210408181153909](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210408181153909-20210408204011324.png)

-   `disable_xp_cmdshell`：关闭`xp_cmdshell`。

-   `xp_cmdshell (cmd)`  ：执行命令。

    ![image-20210408181216647](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210408181216647-20210408204011509.png)

-   `sp_start_job`：使用`sql server`的身份执行命令。（没有回显）

-   `! (cmd)`：在本地执行系统命令。



## 补充说明

mssql的两种认证方式，以及提权（在写了在写了——新建文件夹）