采用C/S星状结构的Linux、unix集中配置系统
协议HTTPS的XMLRPC
XMLRPC是使用http协议作为传输协议的rpc机制，使用xml文本的方式传输数据
原理就是master->client或agent
所有的客户端定期使用facter工具把client的基本配置信息，通过https的XMLRPC协议发送给master，master通过分析client主机名，找到该主机的配置代码，然后编译为ruby，将编译好的伪代码用上面的协议再发回给client，client执行代码完成配置，并且把代码情况反馈给master，这种都是由client请求拉取。还有一种是由master主动推送到client形成分发。分发存在bug锁，用的比较少
工具：facter用于收集主机信息，puppet用于同步

安装配置============
1.避免影响
/etc/init.d/iptables stop
#时间同步----重要
/user/sbin/ntpdate pool.ntp.org

2.ruby环境
yum -y install ruby
3.创建puppet组和用户
groupadd puppet
useradd -g puppet -s /bin/false -M puppet
4.设置hosts----重要！涉及到CA证书，修改后之前的CA证书会无效
hostname查看主机名
修改主机名
vim /etc/sysconfig/network
======================================
NETWORK=yes
NETWORKING_IPV6=yes
HOSTNAME=mfsmaster
======================================
master.test.com
echo "ip地址 主机名">>/etc/hosts	// 主机名可以是IP可以是域名
echo "192.168.15.20 master.test.com">>/etc/hosts
echo "192.168.15.21 agent.test.com">>/etc/host   //客户端的主机名无所谓，服务器端要改

cat /etc/hosts  // ping master

5.安装facter和puppet
##下载地址
##wget http://projects.puppetlabs.com/attachments/download/1101/facter-1.5.8.tar.gz
##wget http://projects.puppetlabs.com/attachments/download/1114/puppet-2.6.1.tar.gz
mkdir -p /tools/puppet
cd /tools/puppet
文件放puppet目录
tar xf facter-1.5.8.tar.gz		// 安装时留意有没有出系统信息，没出会报错
cd facter-1.5.8
ruby install.rb

#check facter
facter

#install puppet
cd /tools/puppet
Tar zxf puppet-2.6.1.tar.gz
cd puppet-2.6.1
ruby install.rb

mkdir -p /etc/puppet	// 还在puppet目录执行
cd conf/redhat/*  /etc/puppet/
cd conf/auth.conf  /etc/puppet/

配置篇
=============master端=============
#建立配置文件目录
mkdir /etc/puppet/manifests -p	// 还在puppet目录执行

#把服务项添加到服务目录，设置自启动
cp /etc/puppet/server.init  /etc/init.d/puppetmaster
chmod 755 /etc/init.d/puppetmaster	// 755应用程序可执行权限
#自启动（可不设置）
chkconfig --add puppetmaster
chkconfig --level 35 puppetmaster on

#启动puppet master
service puppetmaster start
#查看端口：netstat -lntup

#check 8140端口
netstat -lnt
=============agent端=============
client需要master授权才能通信，需要CA证书才允许链接master
1.client请求master授权client再去请求，建立通信授权
puppetd --test --server master.test.com

显示出warning警告是成功的
2.在master端查看有谁在请求证书puppetca -l
在master端单独对谁授权证书puppet -s 域名，puppet -s -a 对所有请求授权


发送了一个证书，等待client取得证书
3.在client端再执行一次puppetd --test --server master.test.com

重置CA证书
client端
cd /var/lib/puppet
rm -rf ssl		// 删除所有ssl连接
master端
Cd /var/lib/puppet/ssl/ca/signed
rm -f agent.test.com.pem
再重新做3步授权步骤

配置管理
