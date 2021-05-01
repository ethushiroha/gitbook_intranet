# getTGT.py



## 用途

对于给定的用户名/密码（明文/hash/aes），向 kdc 请求 TGT 并保存为 `.cache` 格式的文件。



## -h 信息

```shell
C:\opt\htb\intranet\impacket\examples> python3 getTGT.py
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

usage: getTGT.py [-h] [-ts] [-debug] [-hashes LMHASH:NTHASH] [-no-pass] [-k] [-aesKey hex key] [-dc-ip ip address] identity

Given a password, hash or aesKey, it will request a TGT and save it as ccache

positional arguments:
  identity              [domain/]username[:password]

optional arguments:
  -h, --help            show this help message and exit
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

Examples: 
        ./getTGT.py -hashes lm:nt contoso.com/user

        it will use the lm:nt hashes for authentication. If you don't specify them, a password will be asked
```



## 原理

根据Kerberos 协议 AS_REP 部分， 在 KDC 验证了 A 的身份之后，会产生一个 TGT ，是用 KDC 的密码的 hash 加密的消息。

这个 TGT 将被用于 A 与 KDC 之间的认证。（否则每一次请求都需要认证， KDC 的压力很大）



## 使用方法

在已经获取到某一个域用户 A 的 hash/密码 之后使用。



### 小栗子

以hack the box -> start point -> Pathfinder 为例子。

得到了域用户`MEGACORP.LOCAL/sandra`的密码`Password1234!`，定位到域控ip `10.10.10.30`。

`Password1234!` 的 `lm:nt` 为 `aad3b435b51404eeaad3b435b51404ee:29ab86c5c4d2aab957763e5c1720486d` ， 当然是用密码也是可以的。

![image-20210501153911940](https://gitee.com/ethustdout/pic2/raw/master/uPic/image-20210501153911940.png)

由于 TGT 并不都是可见字符， cat 不能很好的显示出来。

`Rubeus` 工具中的 `asktgt` 模块也能够完成请求 TGT 的功能，并且能将其以 base64 的形式打印出来。

