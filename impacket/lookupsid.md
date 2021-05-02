# lookupsid.py



## 什么是sid

sid(`Security Identifiers`)安全标识符，是标识用户、组和计算机帐户的唯一号码。Windows系统（windows 2000以后）会根据sid判断用户的权限。

```
创建帐户 A -> 得到sid_A -> 创建 file_A
删除帐户 A -> 销毁sid_A
再次创建帐户 A -> 得到sid_A_2 -> 无法访问 file_A
```

即：只有sid相同的用户才会被判定为相同用户，并具有相同的访问权限。

假设在机器A上有一个帐户B，在机器C上有个帐户D，且B D 的sid相同（可以使用某些工具修改sid），那么 B D 之间就能互相访问。



### Sid 的组成

sid 的样例如下

使用命令：`whoami /user`可以查看自己的sid

![image-20210411134950788](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411134950788.png)



-   第一部分 `s` 表示这是 sid
-   第二部分 `1` 表示 sid 的版本号是 `1`
-   第三部分 `5` 表示 标识符的颁发机构，对于颁发机构为 `NT`  的，值就是 `5`
-   第四、五、六部分 `xxx-yyy-zzz` 表示一系列的子颁发机构
    -   对于 pc 来说，只有以上六部分
-   第七部分表示帐户和组



对于在同一台 pc 上的不同用户，前面六个部分都是一样的，只有第七部分不一样，通过 msrpc 暴露的接口可以通过枚举 sid 来爆破用户名。



## 用途

当目标开启了，msrpc就可以使用，旨在查找远程用户和组。

不是只有对域控可以使用。



## -h 信息

```shell
C:\opt\htb\intranet\impacket\examples> python3 lookupsid.py -h                                                                                                                                                                                                                                                          1 ⨯
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

usage: lookupsid.py [-h] [-ts] [-target-ip ip address] [-port [destination port]] [-domain-sids] [-hashes LMHASH:NTHASH] [-no-pass] target [maxRid]

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>
  maxRid                max Rid to check (default 4000)

optional arguments:
  -h, --help            show this help message and exit
  -ts                   Adds timestamp to every logging output

connection:
  -target-ip ip address
                        IP Address of the target machine. If omitted it will use whatever was specified as target. This is useful when target is the NetBIOS name and you cannot resolve it
  -port [destination port]
                        Destination port to connect to SMB Server
  -domain-sids          Enumerate Domain SIDs (will likely forward requests to the DC)

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful when proxying through smbrelayx)
```



## 原理

通过[MS-LSAT] MSRPC接口的Windows SID暴力破解程序示例，

有关于`[MS-LAST]`可以看看微软的[官方文档](https://docs.microsoft.com/zh-tw/openspecs/windows_protocols/ms-lsat/678dd7ed-611b-4d94-8f5c-a06c76ae5c7e)。简单来说就是，接受一个sid，返回该sid对应的用户信息。



## 使用方法

突破一台机器，得到域用户的密码，定位到域控之后，想要获取域内其他用户的信息。



### 小栗子

以hack the box -> start point -> Pathfinder 为例子。

得到了域用户`MEGACORP.LOCAL/sandra`的密码`Password1234!`，定位到域控ip `10.10.10.30`。

使用`python3 lookupsid.py MEGACORP.LOCAL/sandra:"Password1234\!"@10.10.10.30`：

![image-20210411140045221](https://gitee.com/ethustdout/pics/raw/master/uPic/image-20210411140045221.png)

其中

-   标记为`SidTypeUser`的为域用户
-   标记为`SidTypeGroup`的为域组
-   标记为`SidTypeAlias`的<font color='red'>暂时未知，如果有知道的师傅，请教教我。</font>

