## 记录一下我失败的ssh学习过程  
> 中英文来回切输入法太麻烦了，我决定尽量用英文写，但我英语很烂，大概很多错误  

### sshd server can use several different authentication methods  
- password(sent securely)  
- RSA and DSA keys  
- Kerberos  
- s/key and SecureID(OTP)  
(the last two are so complex.My teacher did not teach us.And I decide to give up.Beacuse it does not have much connection with my major.)  

### Service Introduction:OpenSSH  
- RPM: openssh,openssh-clients(connect others),openssh-server(be connected)  
- Service: sshd  
- Listening Port: 22/tcp  4.
- Service Configuration File: /etc/ssh/sshd_config  
- Service Startup classifiaction:  
    > Running State start: systemctl start service  
    > Boot from boot: systemctl enable service  
    
Begin 1(Terminal):  
1.install  
2.#systemctl start sshd;systemctl enable sshd(Beacuse I am using root permission, so it's #)  
3.#systemctl status sshd  
4.#vim /etc/sysconfig/selinux (里面非注释有两行，selinux和selinuxtype。他们各有三种取值。如果SELINUX=enforcing，则要执行下面几步。)（#getenforce可获取当前工作方式，#setenforce可以修改。enforcing和permissive可以在不重启的模式下切换，与disabled切换需要重启#reboot）  
5.（更改完成后，找主配置文件）#vim /etc/ssh/sshd_config(想连别人的时候，改ssh_config;别人连我的时候，改sshd_config)  
    （改文件时，当改注释时，先复制要改的内容，Port 22 改为 Port 23456，数字随便）  
6.因为更改了端口，所以先过滤一下 netstat -lntup | grep ：23456 如果无人监听，可以使用这个端口  
7.每改一次配置文件就要重启服务 #systemctl restart sshd 如果报错 #journalctl -xe #semanage port -a -t ssh_port_t -p tcp 23456 放行这个23456端口 （#semanage port -l 是所有端口，#semanage port -l | grep ssh ）  
8.#semanage port -l | grep ssh 此时就可以发现23456端口被放行了，再次重启#systemctl restart sshd  
（如果#getenforce是permissive或者disabled，这几步可以不做）  
9.#firewall-cmd --permanent --add-port=2234/tcp  
10.#firewall-cmd --reload（默认RHEL7使用firewall）  
然后就结束了。  

#### 但是不知道为啥我实验时是不对的（哭死）  
Begin 2(Terminal):  
1.$[user1@A ~]$ssh-keygen(尽量不要用root)(设置私钥四位以上)(私钥与另一台主机无关)  
2.$[user1@A ~]ssh-copy-id user2@IP  
3.$[user2@A ~]ssh user2@IP  

Note:  

     - 在主机A上，通过juser1用户，使用ssh-keygen生成的两个密钥:id_rsa和id_rsa.pub  
     - 私钥（id_rsa）保存在本地主机  
     - 公钥（id_rsa.pub）需要传递到对端B主机  
     - 传输使用的指令 ssh-copy-id
    

