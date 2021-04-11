# GetNPUser.py



## 用途

>    对于域用户，如果设置了选项”Do not require Kerberos preauthentication”，此时向域控制器的88端口发送AS_REQ请求，对收到的AS_REP内容(enc-part底下的ciper，因为这部分是使用用户hash加密session-key，我们通过进行离线爆破就可以获得用户hash)重新组合，能够拼接成”Kerberos 5 AS-REP etype 23”(18200)的格式，接下来可以使用hashcat对其破解，最终获得该用户的明文口令。——参考自https://daiker.gitbook.io/windows-protocol/kerberos/1



## -h 信息

```shell
C:\opt\htb\intranet\impacket\examples> python3 GetNPUsers.py -h                                                
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

usage: GetNPUsers.py [-h] [-request] [-outputfile OUTPUTFILE] [-format {hashcat,john}] [-usersfile USERSFILE] [-ts] [-debug] [-hashes LMHASH:NTHASH]
                     [-no-pass] [-k] [-aesKey hex key] [-dc-ip ip address]
                     target

Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking

positional arguments:
  target                domain/username[:password]

optional arguments:
  -h, --help            show this help message and exit
  -request              Requests TGT for users and output them in JtR/hashcat format (default False)
  -outputfile OUTPUTFILE
                        Output filename to write ciphers in JtR/hashcat format
  -format {hashcat,john}
                        format to save the AS_REQ of users without pre-authentication. Default is hashcat
  -usersfile USERSFILE  File with user per line to test
  -ts                   Adds timestamp to every logging output
  -debug                Turn DEBUG output ON

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials
                        cannot be found, it will use the ones specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  -dc-ip ip address     IP Address of the domain controller. If ommited it use the domain part (FQDN) specified in the target parameter
```



## 原理

对于Kerberos不太了解的师傅们，可以看看[我的博客](https://www.ethushiroha.com/windows_kerberos.html)。写的菜，请轻喷。

在Kerberos的AS验证过程中有KDC验证A身份的过程，如果某个用户启用了`Do not require Kerberos preauthentication`（不需要kerberos预身份验证）选项，那么就会跳过这一步。

这样一来，KDC就会直接把`TGT`和`A_password_hash(KDC_key, date-time)`作为AS_REP发送给A，A得到了`A_password_hash(session Key)`之后，通过暴力破解的方式，得到`A_password`。



## 使用方法

-   对于没有开启`Do not require Kerberos preauthentication`的用户，会返回开启了`Do not require Kerberos preauthentication`的用户列表。

-   对于开启了`Do not require Kerberos preauthentication`的用户，会获得他的TGT和hash加密后的session key。



### 小栗子

以hack the box -> start point -> Pathfinder 为例子。

获得了域内用户`MEGACORP.LOCAL/sandra`的密码，目标`KDC`的ip为`10.10.10.30`。

使用`MEGACORP.LOCAL/sandra`的身份使用`GetNPUser.py`，发现用户`MEGACORP.LOCAL/svc_bes`用户开启了不需要预先认证权限

![image-20210409150151197](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210409150151197.png)

于是使用`svc_bes`的身份再次使用，获得了TGT

![image-20210409162658837](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210409162658837.png)

使用john或者hashcat解密它，得到`svc_bes`用户的密码

![image-20210409163258501](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210409163258501.png)



