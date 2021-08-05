# Valinor_linux_21_8_5  
# 2021.8.5  
*---------------------------Linux指令学习-----------------------*  
Linux文本三剑客  
## grep
## sed
## awk
----------------grep(文本过滤工具)----------------  
`grep [options] pattern [files...]`  
作用:将模式中包含特定字符串的行过滤打印出来  
grep root /etc/passwd          把passwd中包含root的行打印出来  
grep -v root /etc/passwd      把passwd中不包含root的行打印出来  
例：查硬盘分区利用率并排序/查客户端ip  
df |grep /dev/sd |tr -s ' ' % |cut -d% -f5 |sort -nr  
ss -tn |grep ESTAB |tr -s ' ' : |cut -d: -f6 |sort -nr  
例：查当前登陆用户的字符串的行  
grep 'whoami' /etc/passwd     // 以whoami字符串作为搜索条件  
grep $USER /etc/passwd         // 以$USER变量作为搜索条件  
grep "$USER" /etc/passwd      // 识别变量  
grep '$USER' /etc/passwd       // 以$USER字符串作为搜索条件  
PS=''和""的区别：单引号会识别内容为字符串；双引号可以寻找变量  
例：显示字符串和前后三行  
grep -nA3 root /etc/passwd     // 前三行  
grep -nB3 root /etc/passwd     // 后三行  
grep -nC3 root /etc/passwd     // 前后三行  
例：显示多个字符串的并集行  
grep -e root -e wang /etc/passwd  
例：显示多个字符串的交集行  
grep root /etc/passwd |grep wang  
例：读取文件中的字符串，与之匹配并打印出行  
```
cat > xxx.txt  
*****xxx.txt*****  
root  
ctrl+c  
*****xxx.txt*****  
grep -f xxx.txt /etc/passwd    // 实际上这也是取交集  
```
---------------------------------------------------------------  
glob 通配符 对文件的名字  
*  
?  
[abc] a b c某一个字符  
[^abc] 除了a b c的某一个字符  
[0-9]  
---------------------------------------------------------------  
正则表达式 对文件内容  
---------------------------------------------------------------  
nano = 构建文件   cat -E = 在每行结束时加个$  tac = cat反过来  
rev = 可以直接手写，得出反过来的  
more/less = 文件内容的部分展示  
head = 前十行  
cat -n passwd |head -3 = 带有行号的前三行（管道组合命令）  
例：要12个数字字母下划线，其他的不要  
cat /dev/urandom |tr -dc 'a-zA-Z0-9_' |head -c12  
tr命令意为translate替换 -dc=-d删除 -c取反  

tail -n 3 文件名 = tail -n 行数 文件名  
查看文件的倒数3行内容，常用于观察log日志文件，同样可以用管道  
tail -f -n 3 文件名                                                                        tail -F -n 3 文件名  
查看实时刷新的文件的倒数3行内容(文件被删除再恢复后无法监测，跟踪的是文件描述符)        查看实时刷新的文件的倒数3行内容(文件被删除再恢复后可以监测，跟踪的是文件名字)面试常问  

cut -d分隔符 -f列,列 /etc/passwd  
cut -d: -f1,3 /etc/passwd                      cut -d: -f1,3,5-7 /etc/passwd  
抽取以冒号作为分隔符分割出来的第一列和第三列    分割出来的第一列和第三列和第五到七列  
例：抽取分区利用率  
df |cut -c44-46    44-46是因为df出来的列表中，到利用率的第一个数字是43列，从44列开始是利用率第一位数，45列是第二位数，46列是第三位数，因此取的时候就是取这三列  
例：把df中的空格分割换成冒号分割再取不带%的利用率  
df |tr -s ' ' :|cut -d% -f5  
-s指的是压缩，不压缩的话会将连续空格替换成连续冒号，这里其实已经把原来的列重新按新的分隔符分割了  
-f5指的是压缩分割以后，利用率是第5列  

例：把本机的IP取出来----属于通用命令，可以作为备用脚本用(centos6和7对于ifconfig eth0是不一样的，我们要考虑好版本)  
centos7=ifconfig eth0 |head -2 |tail -1 |tr -s ' ' |cut -d' ' -f3  
centos6=ifconfig eth0 |head -2 |tail -1 |tr -s ' ' : |cut -d: -f2 |tr -dc '[0-9].'  
centos5/6=ifconfig eth0 |head -2 |tail -1 |tr -dc '[0-9].' |tr -s ' ' |cut -d' ' -f2  
tr -dc '[0-9].'=删除0-9和小数点以外的所有，因为centos6后有多余的  
------------------指定输出内容的分隔符------------------  
cut -d: -f1,3 /etc/password         这个是输入:输出用:  
cut -d: -f1,3 --output-delimiter=+ /etc/password         这个是输入:输出用+  

纵向合并cat xx.cfg xxx.cfg  
横向合并paste xx.cfg xxx.cfg  
例：
seq 1 10 > xxx1.cfg     按顺序输出1-10替换到xxx1.cfg  
cat xxx.cfg  
echo {a..k}|tr ' ' "\n" > xxx2.cfg     将a-k以换行的形式替换到xxx2.cfg  
paste -d":" xxx1.cfg xxx2.cfg     以:分隔横向合并  
-d"分隔符" 文件名1 文件名2  
paste -s xxx1.cfg xxx2.cfg  
-s显示在一行(默认为多个空格为分隔符)  

---------------------------分析统计工具(wc)---------------------------  
wc xxx1.cfg  
行数 | 单词数 | 字节数 | 文件名 | 最长行
-l=行数 | -w=单词数 | -c=字节数 | -m=字符总数 | -L=显示最长行长度
wc -L xxx1.cfg  
例：统计连接数  
echo 'ss -tn |wc -l'-1|bc  
echo 'xxxxx'-1|bc  
将统计的行数通过运算传回来，这个直接统计有一个标题，所以减一  
例：把连接的ip取出来  
ss -tn |tr -s ' ' : |cut -d: f6 |tr -d '[[:alpha:]]'      [[:alpha:]]=英文字母  
ss -tn |tr -dc '0-9 .\n:' |tr -s " " : |cut -d: -f6  
例：把一个文件里的单词全部提取出来  
cat /etc/profile |tr -sc 'a-zA-Z' '\n' |wc -l  
tr -sc 'a-zA-Z' '\n'压缩，除了字母以外，都替换成\n换行  

----------------------分析统计工具(sort)---------------------  
sort [参数] 文件名  文本排序  
sort -t: -k3 -n /etc/passwd    用:作为分隔符对第3列进行数字正向排序  
sort -t: -k3 -nr /etc/passwd  
-r  倒序排序  
-u 唯一，重复项合并  
-R Random随机排序  
例：取某个分区利用率大于80  
df |tr -s ' ' % |cut -d% -f5|sort -nr |head -n1  
df |tr -s ' ' % |cut -d% -f5|sort -n |tail -n1  
例：统计log有哪些ip访问  
cut -d " " -f1 /var/log/httpd/access_log |sort -u |wc -l  
例：将远程连接的ip取出来统计  
ss -tn |tr -s ' ' : |cut -d: -f6 |sort -u |tr -d '[[:alpha:]]' |wc -l  

----------------------分析统计工具(uniq)---------------------  
删除文件中相邻上下行的重复内容  
例：查看log中前十个访问最多的ip访问连接了多少次(面试题)  
cut -d " " -f1 /var/log/httpd/access_log |sort |uniq -c |sort -nr |head  
因为这个日志中的ip不全是相邻上下行，所以用sort先排序，保证上下行是一样的  
-c=显示重复的次数  
例：查看访问最多的客户端ip访问连接了多少次  
ss -tn |tr -s ' ' : |cut -d: -f6 |sort |uniq -c |sort -nr |head  
例：取出两个文件内容的[交集][不一样的][并集](面试题)  
cat xxx1.cfg xxx2.cfg |sort |uniq -d  
cat xxx1.cfg xxx2.cfg |sort |uniq -u  
cat xxx1.cfg xxx2.cfg |sort -u  

----------------------比较工具(diff)---------------------  
```
cat f1
aaa
bbb
ccc
cat f2
aaa
ddd
eee
ccc
diff -u f1 f2     -u=显示详情
****************输出*****************
---f1 时间戳
+++f2 时间戳
@@ -1,3 +1,4 @@      // -指f1;1,3指范围1-3行;+指f2;1,4指范围1-4行
  aaa
 -bbb
+ddd
+eee
  ccc
****************输出*****************
```
实际上我们可以通过指令对文件做备份，误删可以用备份来还原  
diff -u f1 f2 > diff.log  
rm -f f2  
patch -b f1 diff.log   // 指的是将diff.log中的f2还原并命名为f1，原来的f1备份并改名为f1_orig；-b=backup  



P15
XFTP，文件传输工具
SFTP协议，选项内编码是utf-8

reboot：重启Linux

P16
Linux内置vi文本编辑器
1.一般模式：vim打开一个文件就直接进入一般模式，在这个模式中，可以使用【上下左右】移动光标，可以使用【删除字符】【删除整行】【复制粘贴】
2.插入模式：一般模式进入插入模式，按下i，I，o，O，a，A，r，R任意字母
3.命令行模式：读取，存盘，替换，离开vim，显示行号等。按：或者/ +关键字
gg文件头，shirt + g文件末尾

在终端命令行
			i —— 编辑模式 —— ESC键 —— 一般模式
vim 文件  ——  一般模式
			: 命令行模式 —— ESC键 —— 一般模式
一、编写一个Hello.java程序
vim Hello.java --用vim编写一个Hello.java
i --底部变成  -- 插入 --
public class Hello{
	public static void main(String[] args){
		System.out.println("hello,world");
	}
}
按ESC	-- 退出插入模式
:wq	--输入冒号wq，写入并退出
:q	--退出不保存
:q!	--强制退出不保存

P17vim快捷键（在一般模式下输入）
yy：拷贝当前行。5yy：拷贝当前行向下5行。p：粘贴。
dd：删除当前行。5dd：删除当前行向下5行。
查找某个单词：/+关键字，比如查找world，输入/world光标会定位，输入n为下一个
:set nu：设置行号。:setnonu：取消行号。
在一般模式光标定位最末行[G]最受行[gg]
在一般模式下光标定位 行号 + shift + g
在一般模式撤销刚刚输入的内容，按下u


3.查询用户信息指令
id 用户名
查询root用户：id root
su - 切换用户名
高权限切换到地权限不需要密码，反之需要。用exit/logout注销
创建用户jack，指定密码，然后切换到jack
useradd jack
passwd jack
jack
jack
su - jack
su -root
root
logout   -- jack
exit  --root
查询当前用户
whoami/who am i   -- 后者显示第一次登陆的用户
clear --清楚当前命令行记录

4.添加用户组
groupadd 组名
groupdel 组名
增加用户zwj时直接指定组，不指定组则自己一个组
groupadd wudang
useradd -g wudang zwj  |  useradd -g 组名 用户名
修改用户组
groupadd mojiao
usermod -g mojiao zwj  |  usermod -g 要修改组名 被修改用户名

用户和组相关文件
vim /etc/passwd：用户的配置文件，记录用户的各种信息
每行的含义：用户名:口令:用户标识号:组标识号:注释:主目录:登陆shell(bash=bashell)
vim /etc/shadow：口令的配置文件。可以看到加密口令
vim /etc/group：组的配置文件。

P25---------------运行级别---------------
0-关机
1-单用户【找回丢失密码】
2-多用户没有网络服务
3-多用户有网络服务
4-系统未使用保留给用户
5-图形界面
6-系统重启
输入init [0123456]
输入init 运行级别数

指定运行级别：centos7以前  /etc/inittab
如今简化如下：
multi-user.target:analogous to runlevel 3  -- 多用户类似于运行级别3
graphical.target:analogous to runlevel 5    -- 图形化类似于运行级别5

#To view current default target,run:
systemctl get-default     -- 查看当前运行级别，系统ctl获取当前默认值

#To set a default target,run:
systemctl set-default TARGET.target    -- 更改默认运行级别，系统ctl设置默认值target
systemctl set-default multi-user.target/graphical.target

P26---------------找回root密码---------------
面试题：如何找回root密码
重启Linux进入centos系统前按“e”进入编辑模式
编辑模式中，用方向键移动光标，找到“Linux16”开头所在行
在行尾部输入：init=/bin/sh
然后按“CTRL+X”，进入单用户模式
来到新的命令行输入：mount -o remount,rw /
输入passwd
回车出现以下样式
@@@@ root @@@@
@@@@@输入密码【1234567890】-----回车，再次输入【1234567890】回车
出现passwd@@@。。。。代表密码修改成功，会重新进入到命令行中
输入touch /.autorelabel
输入exec /sbin/init
-----------等-----------

P27---------------帮助指令---------------
man(获得指令的帮助信息)
man ls   -- 指令描述，退出帮助按下q
-a  列出所有文件，包括隐藏文件(以"."开头的都是隐藏文件)
-l   单列输出
ls -la /root或 ls -al  /root列出该目录下所有文件并进行单列输出，参数可以组合

help指令(shell内置命令的帮助信息)
help cd   -- 描述cd用法


P28








----------%%基本正则表达式%%----------  
基本正则表达式：BRE  
扩展正则表达式：ERE  grep -E, egrep  
引擎：PCRE(是一个Perl库,包括 perl 兼容的正则表达式库)  
字符匹配、匹配次数、位置锚定、分组  |  帮助指令man 7 regex  
一、字符匹配  
\. 匹配任意单一字符   grep "r..t" /etc/passwd  —> 以r开头t结尾的单词被过滤  
[] 匹配内容中指定范围内的任意单一字符  
ifconfig eth0 |grep netmask |grep '[[:digit:].]'    —>过滤ip信息中的ip地址  
'[[:digit:].]' = 变量用单引号识别，[:digit:].是数字和小数点,等价于[0-9.]  
参数：[^a]=指定范围内除了a字符以外的任意单一字符  

二、匹配次数  
\* 匹配任意次，包括0次，这种遇到符合条件的会一直匹配俗称贪婪模式  
```
cat > f1
goooogle
google
gogle
ggle
```
例：查两个o以上，过滤goo，因为o*可以是0次  
grep "gooo*gle" f1   —>返回goooogle、google  
例：grep "g.*gle" f1   任意字符任意匹配  
\\? 匹配前面的字符0或1次  
例：grep "go\?gle" f1  —>返回gogle、ggle  
\\+ 匹配前面的字符至少1次  
例：grep "go\+gle" f1  —>返回goooogle、google、gogle  
\\{x\} 匹配前面的字符精确次数  
例：grep "go\{4\}gle" f1   —>返回goooogle  
例：grep "go\{2,4\}gle" f1   —>返回goooogle、google  
例：grep "go\{2,\}gle" f1   —>返回2个以上，上不封顶  
例：grep "go\{,4\}gle" f1   —>返回4个以下，下不设限  
例：将ifconfig的ip地址过滤出来  
ifconfig eth0 |grep "[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}"  
返回192.168.xx.x、255.255.255.0、192.168.xx.x  
[0-9]\{1,3\}. = 0-9的数字匹配三次后有小数点，相当于三位数带小数点；上面\.是因为只输入.会被认为是匹配任意单一字符，用\转义被认为就是点  

三、位置锚定  
^ 行首锚定，用于模式的最左侧  grep "^root" /etc/passwd  
$ 行尾锚定，用于模式的最右侧  grep "bash$" /etc/passwd  
grep "^gogle$" /etc/passwd  仅返回gogle  
非空的行：grep -v "^$" /etc/passwd    ^$=空行(回车行)  -v反选  "^[[:space:]]*$" = 空白行  
例：查df硬件利用率行  
df |grep "^/dev/sd"  
\\< 或 \b 词首锚定，用于单词模式的最左侧  grep "\<root" /etc/passwd  
\\> 或 \b 词尾锚定，用于单词模式的最右侧  grep "root\>" /etc/passwd  
完整单词：grep "\<root\>" /etc/passwd  
只匹配root完整单词：grep -o "\broot\b"   -o=只匹配字符串参数  

四、分组  
1.语法：\(root\) root为匹配字符串  
```
cat > f1
abcabc
abcabcabc
abc
grep "\(abc\)\{3\}" f1  (abc)作为一个整体匹配过滤三次，返回abcabcabc
```
2.参数：  
\\1  从第一个左括号以及与它搭档的右括号之间匹配的字符;  
\\2  从第二个左括号以及与它搭档的右括号之间匹配的字符  
不匹配模式本身，只匹配字符！  
例：匹配abc之后还有abc的行  
```
cat > f1
abcabc
abcabcabc
abc
abccc
abc abc
abc
cat f1 |grep "\(abc\).*\1"  f1   返回abcabc、abcabcabc、abc abc
\1=abc
```
例：匹配2个字符串循环的行  
```
cat > f1
xyz xyz
abc xyz abc xyz
cat f1 |grep "\(abc\).*\(xyz\).*\2"   \2=xyz，返回abc xyz abc xyz
```
例：匹配模式还是匹配字符？  
```cat > f1
123ab123xxy123
234xxx567
cat f1|grep "\([0-9]\)\{3\}.*\1"   返回123ab123xxy123
为什么不返回234xxx567？------因为\1=[0-9]=数字本身，123后面也是123
234后面也必须是234而不是567
3.或
语法：\|
grep "^a\|b" /etc/passwd   过滤以a开头或有b的行
grep "^\(a\|b\)" /etc/passwd  过滤以a或b开头的行
```
练习题-----  
取ifconfig的ip-----ifconfig |grep -o "\([0-9]\{1,3\}\.\)\{3\}[0-9]\{1,3\}" |head -1  
显示/proc/meminfo中以大小s开头的行（两种方法）  
显示/etc/passwd中不以/bin/bash结尾的行  
显示用户rpc默认的shell程序  
找出/etc/passwd中的两位或三位数  
显示centos7的/etc/grub2.cfg中，至少以一个空白字符开头且后面有非空白字符的行  
找出 "netstat -tan" 中以LISTEN后跟任意多个空白字符结尾的行  
显示centos7所有用户名和uid  
添加用户bash、testbash、basher、sh、nologin(shell为/sbin/nologin)，找出/etc/passwd用户名和shell同名的行  
利用df和grep，取出磁盘分区利用率，从大到小排序  
在文件中用正则过滤出qq号、身份证号、手机号、邮箱  
----------%%扩展正则表达式%%----------  
egrep = grep -E  
egrep [options] pattern [file...]  
字符匹配：.任意单个字符、[]指定范围单一字符、[^]指定范围以外单一字符  
次数匹配：* 匹配前面字符任意次  
	? 匹配前面字符0次或1次  
	+ 匹配前面字符1次或多次  
	{m} 匹配前面字符m次  
	{m,n} 匹配前面字符至少m次，至多n次  
位置锚定：^ 行首  
	  $ 行尾  
	  \< , \b 词首  
	  \> , \b 词尾  
分组：() 后向引用：\1 , \2 ...  
a | b          a或b  
C | cat       C或cat  
(C | c)at     Cat或cat  

练习题-----  
1.使用egrep取出 /etc/rc.d/init.d/functions 中其基名和目录名  
基名指的是文件名的基础名  basename /etc/rc.d/init.d/functions  
echo "/etc/rc.d/init.d/functions" |grep -Eo "[^/]+$" 返回functions  
echo "/etc/rc.d/init.d/functions" |grep -o "[^/]\+$"  --基础正则  
[^/]+$ = 不带/且匹配多次的行尾  
--目录名(后面的sed可以一次取完)  
echo "/etc/rc.d/init.d/functions" |grep -Eo ".*[^/]" |grep -Eo ".*/"  
.*[^/] = 过滤出任意单一字符任意匹配次数且最后不带/  --为什么最后要不带/,是为了规避/etc/rc.d/init.d/这种情况  
.*/ = 过滤出任意单一字符任意匹配次数且最后带/  

2.取ifconfig的ip地址  
0-9,10-99 [1-9]?[0-9]  
100-199 1[0-9]{2}  
200-249 2[0-4][0-9]  
250-255 25[0-5]  
```
ifconfig | grep -Eo "(([1-9]?[0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([1-9]?[0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])"   --参数里面的|前后不能加空格
```
------------%Vim%------------  











---练习题---  
Linux启动过程，NFS原理，RSYNC，RAID0-5区别，LVS模式，MYSQL主从同步，防火墙DNAT，SNAT，备份恢复方案，监控  
把一个目录中大于100k的文件移动到另一个目录下边  
取一个文件的第1到第200行  
遭受木马攻击，每个文件都被打入了一串js字符串，怎么恢复  
有个文件10000多行，提取前5000行，把里面的xxx替换成sss  
写一个脚本批量建立100个用户  
CDN全程，原理  
高级：一个512M的分区，不停写入1K大小的文件，请问可以写多少个文件？描述其限制原因及解决方法  



Email
---------------------------------------------------------------------------------------
在XX上看到您发的贴子，特来自荐。
我自信能符合贵公司大部分要求，并可以尽快学习达到符合公司未来要求之目的。
我没有在比较大型的门户网站写博客的习惯，技术文档以及收获比较习惯整理在www.ssss.com，这是因为我与这个站长有些交情，在这个站点上整理一些东西。如果您方便访问那个站点，可以看这个连接，里面是我最近整理的技术内容。
我自认为在我以往的工作中，我的职业素养还是可以的，甚至能让我现在公司领导给贵公司写推荐信。
最后，再次表达我对您和贵公司的的敬意，希望能够得到您的答复。
---------------------------------------------------------------------------------------
公司的运维部门的人员和服务器大概是什么样的规模
介绍完以后---开始介绍架构，集群---话题引到自己熟悉的领域减少突然发问的频率（大厂基本无用）
要名片，然后技术面完就把问题答案发回去，可能会有面试官恢复
-------------------------------------
我觉得我的能力值8K，因为我也和身边坐运维的聊过，水平稍弱于我的也拿到了8K，对于这个薪水我不是特别在意，我比较在意公司的发展和个人的经验提升
-----------------------------------------------
做技术不仅需要有独立思考，搜集信息的能力，还要有钻研攻坚的意志，我都在加强这些要素。在刚毕业时，我选择在xx工作，开始接触企业一线的生产环境，不断摸索，在而后的大型项目中根据领导的意见和别的同事通力合作，很好的完成了自己负责的模块。有了这两年的沉淀，我相信可以试着接触更好更深入的技术线。但我毕竟也只有两年的经验，没有能够得到一个独当一面的机会，因此我希望这次换工作能成为一次飞跃，并珍惜利用好这个机会。
xxxx是一个稳中求进的好企业，员工只需要做好自己分内的工作，按部就班发挥自己的作用就足够了。我现在负责的项目按照计划，已经步入了相对稳定的运维阶段，变更改善的措施都显得较为保守，因此我觉得是时候跳出舒适圈。
我很少参加面试，对目前的薪资水平不会太了解。如果贵公司认为我配不上我所提的薪资待遇，我可以降低待遇要求。
如果能在您手下供职，我会把我的能力展示给您看，也把我的学习能力展示给您看。我以前的公司都用加薪和提升待遇肯定了我的能力，我也相信靠我的努力可以赢得回报
