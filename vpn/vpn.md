#ubuntu14.04搭建vpn方法  
ubuntu14.04搭建vpn，非常简单。首先目前常用vpn协议有三个： 

pptp   应用最广泛，各种平台都能原生支持，速度快。  
l2tp    安全性更高，各种平台都能支持。  
openvpn  linux支持最好，其他平台都需要第三方软件才能使用。安全性高，灵活性强。  
ubuntu 14.04搭建pptp协议的vpn非常简单，本文参考https://help.ubuntu.com/community/PPTPServer  

第一步，在源上安装pptp server  
sudo apt-get install pptpd  

配置pptpd 服务器的ip和客户端的ip段  
sudo vim /etc/pptpd.conf  

跳转到文件最下边，修改以下两行  
localip 192.168.0.1  
remoteip 192.168.0.100-200  

配置pptp server的dns  
sudo vim /etc/ppp/pptpd-options  

找见以下两行，取消注释。如果没有这一步操作，有可能导致无法浏览网页。  
ms-dns 8.8.8.8  
ms-dns 8.8.4.4  

 配置vpn的用户名密码  
sudo vim /etc/ppp/chap-secrets  

内容分为四列，第一列是你自定义的用户名，第二列是服务名，这里填pptpd，第三列是自定义的密码，第四列是允许的ip，这里填写*就可以。字段之间用空格或者TAB隔开。配置多个用户可以添加多行。  
# client      server     secret         IP addresses  
username      pptpd      'password'       *  

设置允许ipv4协议重定向  
 sudo vim /etc/sysctl.conf  
 
取消注释下边这一行  
net.ipv4.ip_forward=1  

 给iptables添加重定向规则  
sudo vim /etc/rc.local  

在return 0 之前添加以下两行  
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE  
iptables -A FORWARD -p tcp --syn -s 192.168.0.0/24 -j TCPMSS --set-mss 1356  
iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356  

重启服务器  
到此，配置完成。  
