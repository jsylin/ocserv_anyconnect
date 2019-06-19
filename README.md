# ocserv_anyconnect
install ocserv_anyconnect  

watch 流程.txt文件  


！！！！！！注意防火墙的开启    
    
首先我们打开 防火墙的NAT：

我们需要先查看我们的主网卡是什么：  
  
ifconfig  
输出结果可能如下，那么我们的主网卡默认是 eth0 ，如果你是 OpenVZ，那么主网卡默认是 venet0 。  
  
如果是 Debian9 系统，则默认网卡名为 ens3，CentOS Ubuntu 最新版本的系统可能为 enpXsX(X代表数字或字母)。  
  
# === 服务器输出示例 === #  
root@debian:~# ifconfig  
eth0      Link encap:Ethernet  HWaddr xx:xx:00:00:a8:a0    
          inet addr:xxx.xxx.xxx.xxx  Bcast:xxx.xxx.225.255  Mask:255.255.255.0  
          inet6 addr: xxx::xxx:ff:fe00:xxx/64 Scope:Link  
          ......  
   
lo        Link encap:Local Loopback    
          inet addr:127.0.0.1  Mask:255.0.0.0  
          inet6 addr: ::1/128 Scope:Host  
          ......  
# === 服务器输出示例 === #


   
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
# 如果是 OpenVZ就执行下面这个：  
iptables -t nat -A POSTROUTING -o venet0 -j MASQUERADE  
然后我们需要打开 ipv4防火墙转发：  
  
echo -e "net.ipv4.ip_forward=1" >> /etc/sysctl.conf && sysctl -p   
最后，我们就需要开放防火墙端口了，教程中示例文件的默认TCP/UDP端口都是 443，如果你改了 那么改为其他端口即可。  
  
iptables -I INPUT -p tcp --dport 443 -j ACCEPT  
iptables -I INPUT -p udp --dport 443 -j ACCEPT  
# 如果你以后要删除规则，那么把 -I 改成 -D 即可。  
iptables -D INPUT -p tcp --dport 443 -j ACCEPT  
iptables -D INPUT -p udp --dport 443 -j ACCEPT  
配置防火墙开启启动读取规则  
一般默认 iptbales 关机后并不会保存规则，这样开机后 防火墙规则也全都清空了，所以需要设置一下。  
   
Debian/Ubuntu 系统：  
  
iptables-save > /etc/iptables.up.rules  
echo -e '#!/bin/bash\n/sbin/iptables-restore < /etc/iptables.up.rules' > /etc/network/if-pre-up.d/iptables  
chmod +x /etc/network/if-pre-up.d/iptables   
以后需要保存防火墙规则只需要执行：   
  
iptables-save > /etc/iptables.up.rules  
  
  
！！！！！！！以及4个证书的生成放置；  
   
   自签SSL证书  
首先在当前目录新建一个文件夹（要养成不把文件乱扔的习惯）。  
  
mkdir ssl && cd ssl  
生成 CA证书  
然后用下面的命令代码（8行一起复制一起粘贴一起执行），其中的两个 doubi 可以随意改为其他内容，不影响。  
  
echo -e 'cn = "doubi"  
organization = "doubi"  
serial = 1  
expiration_days = 365  
ca  
signing_key  
cert_signing_key  
crl_signing_key' > ca.tmpl  
然后我们生成证书和密匙：  
  
certtool --generate-privkey --outfile ca-key.pem  
certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem  
生成 服务器证书  
继续用下面的命令代码（6行一起复制一起粘贴一起执行），其中的 1.1.1.1 请改为你的服务器IP，而 doubi 可以随意改为其他内容，不影响。  
  
echo -e 'cn = "1.1.1.1"  
organization = "doubi"  
expiration_days = 365  
signing_key    
encryption_key  
tls_www_server' > server.tmpl  
然后我们生成证书和密匙：  
  
certtool --generate-privkey --outfile server-key.pem  
certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem  
最后我们在ocserv的目录中新建一个ssl文件夹用于存放证书。  
  
mkdir /etc/ocserv/ssl  
把刚才生成的证书和密匙都移过去：  
  
mv ca-cert.pem /etc/ocserv/ssl/ca-cert.pem  
mv ca-key.pem /etc/ocserv/ssl/ca-key.pem  
mv server-cert.pem /etc/ocserv/ssl/server-cert.pem  
mv server-key.pem /etc/ocserv/ssl/server-key.pem  
现在刚才在当前文件夹新建的 ssl 文件夹就没用了，你可以删除它： cd .. && rm -rf ssl/  
  稍微最后稍微注意一下服务的下载运用    
