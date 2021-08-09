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
