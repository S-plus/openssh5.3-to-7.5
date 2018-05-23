# CentOS6.6下 Openssh 5.3升级7.5  
## 一、安装gcc, pam-devel, zlib-devel, openssl-devel
安装gcc，zlib-devel, pam-devel
验证是否已安装，如果有可跳过
```
rpm –qa|grep gcc

rpm -ivh  mpfr-2.4.1-6.el6.x86_64.rpm
rpm -ivh  cpp-4.4.7-16.el6.x86_64.rpm
rpm -Uvh  libgomp-4.4.7-16.el6.x86_64.rpm
rpm -Uvh  libgcc-4.4.7-16.el6.x86_64.rpm
rpm -ivh  ppl-0.10.2-11.el6.x86_64.rpm
rpm -ivh  cloog-ppl-0.15.7-1.2.el6.x86_64.rpm

rpm -ivh  gcc-4.4.7-16.el6.x86_64.rpm
```
到此安装结束如需安装gcc++
```
rpm -ivh  gcc-c++-4.4.7-16.el6.x86_64.rpm 
```

安装openssl-devel
验证同上
```
rpm -ivh keyutils-1.4-5.el6.x86_64.rpm
rpm -ivh keyutils-libs-1.4-5.el6.x86_64.rpm
rpm -ivh keyutils-libs-devel-1.4-5.el6.x86_64.rpm
rpm -ivh krb5-libs-1.10.3-65.el6.x86_64.rpm
rpm -Uvh libkadm5-1.10.3-65.el6.x86_64.rpm
rpm -ivh krb5-workstation-1.10.3-65.el6.x86_64.rpm
rpm -Uvh libss-1.41.12-22.el6.x86_64.rpm e2fsprogs-1.41.12-22.el6.x86_64.rpm libcom_err-1.41.12-23.el6.x86_64 e2fsprogs-libs-1.41.12-22.el6.x86_64.rpm
rpm -Uvh libss-devel-1.41.12-22.el6.x86_64.rpm libcom_err-devel-1.41.12-23.el6.x86_64 
rpm -ivh e2fsprogs-devel-1.41.12-22.el6.x86_64.rpm
rpm -Uvh libselinux-2.0.94-7.el6.x86_64.rpm libselinux-utils-2.0.94-7.el6.x86_64.rpm libselinux-python-2.0.94-7.el6.x86_64.rpm
rpm -Uvh libsepol-2.0.41-4.el6.x86_64.rpm
rpm -Uvh libsepol-devel-2.0.41-4.el6.x86_64.rpm
rpm -Uvh libselinux-devel-2.0.94-7.el6.x86_64.rpm
rpm -ivh krb5-devel-1.10.3-57.el6.x86_64.rpm
rpm -ivh zlib-1.2.3-29.el6.x86_64.rpm
rpm -ivh zlib-devel-1.2.3-29.el6.x86_64.rpm
rpm -ivh openssl-1.0.1e-57.el6.x86_64.rpm
rpm -ivh openssl-devel-1.0.1e-57.el6.x86_64.rpm
```
注：rpm包版本依具体情况而定
***
## 二、安装telnet服务
```
rpm -ivh xinetd-2.3.14-40.el6x86_64.rpm
rpm -ivh telnet-0.17-48.el6.x86_64.rpm
rpm -ivh telnet-server-0.17-48.el6.x86_64.rpm
```
安装好后
```
vi /etc/xinetd.d/telnet
```
将其中disable字段的yes改为no以启用telnet服务 
```
mv /etc/securetty /etc/securetty.old    //允许root用户通过telnet登录 
service xinetd start                    //启动telnet服务 
chkconfig xinetd on                    //使telnet服务开机启动，避免升级过程中服务器意外重启后无法远程登录系统
```
修改防火墙配置
```
vim /etc/sysconfig/iptables
```
在其中添加telnet的端口，默认为23
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23 -j ACCEPT
```
重启防火墙服务
```
service iptables restart
```
测试telnet能否正常登入系统  
***
## 三、升级openssh
### 1. 备份当前openssh
```
mv /etc/ssh /etc/ssh.old 
mv /etc/init.d/sshd /etc/init.d/sshd.old
```
### 2. 卸载当前openssh
```
rpm -qa | grep openssh 
openssh-clients-5.3p1-104.el6.x86_64 
openssh-server-5.3p1-104.el6.x86_64 
openssh-5.3p1-104.el6.x86_64 
openssh-askpass-5.3p1-104.el6.x86_64 
rpm -e --nodeps openssh-5.3p1-104.el6.x86_64 
rpm -e --nodeps openssh-server-5.3p1-104.el6.x86_64 
rpm -e --nodeps openssh-clients-5.3p1-104.el6.x86_64 
rpm -e --nodeps openssh-askpass-5.3p1-104.el6.x86_64 
rpm -qa | grep openssh
```
> 注意：卸载过程中如果出现以下错误
```
[root@node1 openssh-7.5p1]# rpm -e --nodeps openssh-server-5.3p1-104.el6.x86_64  
error reading information on service sshd: No such file or directory 
error: %preun(openssh-server-5.3p1-104.el6.x86_64) scriptlet failed, exit status 1 
```
解决方法： 
```
rpm -e --noscripts openssh-server-5.3p1-104.el6.x86_64
```
### 3. openssh安装前环境配置
```
install -v -m700 -d /var/lib/sshd 
chown -v root:sys /var/lib/sshd
```
当前系统sshd用户已经存在的话以下不用操作 
```
groupadd -g 50 sshd 
useradd -c 'sshd PrivSep' -d /var/lib/sshd -g sshd -s /bin/false -u 50 sshd
```
### 4. 解压openssh_7.5p1源码并编译安装
```
tar -zxvf openssh-7.5p1.tar.gz 
cd openssh-7.5p1 
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords --with-pam --with-zlib --with-openssl-includes=/usr --with-privsep-path=/var/lib/sshd 
make 
make install
```
> 如果在configure openssh时，如果有参数 –with-pam，会提示：
  ```
  PAM is enabled. You may need to install a PAM control file for sshd, otherwise password authentication may fail. Example PAM control files can be found in the contrib/subdirectory
  ```
  就是如果启用PAM，需要有一个控制文件，按照提示的路径找到contrib/redhat/sshd.pam，并复制到/etc/pam.d/sshd

### 5. openssh安装后环境配置
在openssh编译目录执行如下命令 
```
install -v -m755    contrib/ssh-copy-id /usr/bin 
install -v -m644    contrib/ssh-copy-id.1 /usr/share/man/man1 
install -v -m755 -d /usr/share/doc/openssh-7.5p1 
install -v -m644    INSTALL LICENCE OVERVIEW README* /usr/share/doc/openssh-7.5p1 
ssh -V              //验证是否升级成功
```

### 6. 启用OpenSSH服务
在openssh编译目录执行如下目录 
```
echo 'X11Forwarding yes' >> /etc/ssh/sshd_config 
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config  #允许root用户通过ssh登录 
cp -p contrib/redhat/sshd.init /etc/init.d/sshd 
chmod +x /etc/init.d/sshd 
chkconfig  --add  sshd  # 加入系统启动服务
chkconfig  sshd  on 
```
查看系统启动服务是否增加该项
```
chkconfig --list |grep sshd

sshd               0:off    1:off    2:on    3:on    4:on    5:on    6:off 
```
重启openssh服务
```
service sshd restart
```
> 注意：如果升级操作一直是在ssh远程会话中进行的，上述sshd服务重启命令可能导致会话断开并无法使用ssh再行登入（即ssh未能成功重启），此时需要通过telnet登入再执行sshd服务重启命令。  

### 7. 允许root用户远程登录
```
cp sshd_config /etc/ssh/sshd_config
```
> **vim /etc/ssh/sshd_config 修改 PermitRootLogin yes,并去掉注释配置允许root用户远程登录**  
这一操作很重要！很重要！很重要！重要的事情说三遍，因为openssh安装好默认是不执行sshd_config文件的，所以即使在sshd_config中配置允许root用户远程登录，但是不加上这句命令，还是不会生效！
```
vim /etc/init.d/sshd
```
在   
```$SSHD $OPTIONS && success || failure```  
这一行上面加上一行   
```OPTIONS="-f /etc/ssh/sshd_config"``` 
保存退出
### 8.重启系统验证没问题后关闭telnet服务并将防火墙配置修改回来
```
mv /etc/securetty.old /etc/securetty 
chkconfig  xinetd off 
service xinetd stop
vim /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23 -j ACCEPT //将这条删除
```
重启防火墙服务
```
service iptables restart
```
如需还原之前的ssh配置信息，可直接删除升级后的配置信息，恢复备份。 
```
rm -rf /etc/ssh 
mv /etc/ssh.old /etc/ssh
```
