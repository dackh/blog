# 常用文本命令
## awk
awk是处理文本文件的一个应用程序，几乎所有linux系统都自带这个程序。

### 基本用法
```sh
# 格式
awk 动作 文件

awk '{print $0}' all.log
# 打印all.log 当前行数据  $0代表当前行  '{}'表示动作

echo 'this is a log' | awk '{print $0}' 
this is a log
# awk 会根据空格跟制表符，将每一行分成若干部分，依次用$1,$2代表第几个字段。

# 指定分隔符
awk -F ',' '{print $1}' all.log
```
### 变量
`$ + 数字`表示第几个字段，awk还提供了一些变量，`NF`表示当前有多少行，因此`$NF表示最后一个字段`。
`NR`表示第几行。
```sh
awk -F ',' '{print $1, $(NF-1)}' all.log

# 上面代码中，print命令里面的逗号，表示输出的时候，两个部分之间使用空格分隔，如果要原样输出，放在双引号里面。

awk -F ',' '{print NR ") " $1}' all.log
1) xxx
2) xxx
3) xxx
```
其他变量：
- FILENAME : 当前文件名
- FS : 字段分隔符，默认是空格和制表符
- RSRS：行分隔符，用于分割每一行，默认是换行符。
- OFS：输出字段的分隔符，用于打印时分隔字段，默认为空格。
- ORS：输出记录的分隔符，用于打印时分隔记录，默认为换行符。
- OFMT：数字输出的格式，默认为％.6g

### 函数
awk还提供了一些内置函数，方便对原始数据的处理。
- toupper() ：转换为大写
- tolower() ： 转换为小写
- length() : 字符串长度
- substr()：子字符串

```sh
awk -F ',' '{print toupper($1)}' all.log
```
### 条件
```sh
awk '条件 动作' 文件

# 输出包含aa的行
awk -F ',' 'aa {print $1}' all.log

# 输出奇数行
awk -F ',' 'NR % 2 == 1 {print $1}' all.log

# 输出第三行之后
awk -F ',' 'NR > 3 {print $1}' all.log

# 指定指定字段等于指定值
awk -F ',' '$1 == "aa" || $1 == "bb" {print $1}' all.log
```

### if语句
awk还提供了if结构，用于编写复杂的条件。


## grep 
grep (global search regular expreession and print out the line,全局搜索正则表达式并把行打印出来)。

### 基本用法
```sh
grep [options] pattern [file...]

#options 选项，pattern匹配文本、变量或者正则表达式，file表示文件，可以多个

# 匹配包含数字的文本行
grep -E '[1-9]+' all.log
```
#### 选项
常用选项
```sh
-a : 只显示行数
-i : 忽略大小写
-h : 搜索多个文件，不显示文件名前缀
-n : 行号+文本行
-v : 不匹配的文本行
-w : 匹配整个单词
-r : 递归搜索，搜索当前目录跟子目录
-x : 匹配整个文本行
-E : 匹配正则表达式
```
## sed
sed是一种在线编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为"模式空间(pattern space)"，接着用sed命令处理缓冲区的内容，
处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，文件并没有改变。除非使用重定向存储输出。

sed主要用于自动编辑一个或多个文件。

### 基本用法
```sh
sed [option] 动作

# option是选项

# 删除包含url的行
sed '/url/d' all.log
```
### 选项
```sh
-n：仅显示scrpit处理后的结果。
-e<script>: 以选项中的script来处理输入的文本文件。
-f<script文件>：以选项中的script文件来处理输入的文本文件。
-i：直接修改读取文件的内容，而不是输出到终端。
```

### 动作说明
格式：`[n1[,n2]]function`：n1,n2：不见得会存在，一般代表[选择进行动作的行数]。

function：
a：新增，后面接字符串，在下一行出现
c：取代，后面接字符串，替代n1,n2的行
d：删除
i：插入，后面接字符串，在上一行出现
p：列印
s：取代
```
for file in file_list;do zgrep '服务异常查找TLoanDisplayInfoFlow表失败' ${file} | awk -F'|' '{print $5}' |sort|uniq | xargs -I {} zgrep {} gov_data_sync_daemon-2019-12-11.4.log.gz |grep '收到cmq:' | awk -F'收到cmq:' '{print $2}' | awk -F', 来自topic:' '{print "{\"data\":",$1,",\"cmq_topic\":\"",$2,"\"}"}' >>tmp.txt;done

```



# 链接 ln
为某一个文件在另外一个位置创建一个同步的链接，常用参数 `-s`，进行软链接

具体用法：
`ln 源文件 目标文件`

- `-s`：进行软链接
- `-f`：链接时删除已存在的目标文件（路径无效）
- `-v`：打印连接文件的信息


注意：

- 无论软链接跟硬链接，**链接文件跟源文件保持同步**。
- 软连接在选定位置生成一个镜像，不占用磁盘空间，硬链接在选定位置生成一个和源文件大小相同的文件。

### 硬链接
硬链接指通过索引节点来进行连接，在linux的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号`Inode  Index`。在linux中，多个文件指向同一个索引节点是存在的。这个连接就是硬链接。

硬链接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬链接到重要文件，以防止“误删”功能。目录的索引节点有一个以上的连接，只有当最后一个链接被删除后，文件的数据块及目录的链接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬链接均被删除。

- 不允许给目录创建硬链接
- 只有同一文件系统、`root`用户才能建立硬链接


### 软链接
软链接文件类似`windows`系统的快捷方式，实际上是一个特殊的文件。

在符号链接中，文件实际上是一个文本文件，其中包含的另一文件的位置信息。

- 软链接文件保存源文件的路径信息，当文件从一个目录移到其他目录中，此时在访问连接文件，系统将找不到文件。

# 查找命令
### find
`find`是最强大和最常见的查找命令，你可以用它找到任何你想找的文件

**命令格式：`$ find <指定目录> <执行条件> <指定动作>`**
 
如果什么参数都不加，`find`默认搜索当前目录及其子目录，并且不过滤任何结果（也就是返回所有文件），将他们全部显示在屏幕上。

例子：


	[root@localhost /]# find . -name "docker"
	./run/docker
	./sys/fs/cgroup/cpuset/docker
	./sys/fs/cgroup/net_cls/docker
	./sys/fs/cgroup/hugetlb/docker
	./sys/fs/cgroup/freezer/docker
	./sys/fs/cgroup/devices/docker
	./sys/fs/cgroup/memory/docker
	./sys/fs/cgroup/perf_event/docker
	./sys/fs/cgroup/blkio/docker
	./sys/fs/cgroup/cpu,cpuacct/docker
	./sys/fs/cgroup/systemd/docker
	./etc/docker
	./var/lib/docker
	./usr/bin/docker
	./usr/share/bash-completion/completions/docker

### locate
`locate`命令其实是`find -name`的另一种写法，但是要快得多，原因在于它不搜索具体目录，而是搜索一个数据库`/var/lib/locatedbd`，centos下是`/var/lib/mlocate/mlocate.db`，这个数据库中含有本地所有文件信息。

Linux系统自动创建这个数据库，并且每天更新一次，所以使用locate命令查找最近变动的文件时，先使用`updatedb`命令手动更新数据库。

例子：

	[root@localhost /]# locate docker.
	/etc/selinux/targeted/modules/active/modules/docker.pp
	/usr/lib/systemd/system/docker.service
	/usr/lib/systemd/system/docker.socket
	/usr/share/fish/vendor_completions.d/docker.fish
	/usr/share/man/man1/docker.1.gz


### whereis
 `whereis`命令只能用于程序名的搜索，而且只搜索二进制文件

例子：

	[root@localhost /]# whereis docker
	docker: /usr/bin/docker /etc/docker /usr/share/man/man1/docker.1.gz

### which
`which`命令是在`PATH`变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用`which`命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

例子：
	
	[root@localhost /]# which docker
	/usr/bin/docker

### type
`type`命令用来区分某个命令是shell自带还是外部独立二进制文件提供的。

例子：

	[root@localhost /]# type cd
	cd is a shell builtin
	[root@localhost /]# type docker
	docker is hashed (/usr/bin/docker)




# 进程和网络管理
### ps
`ps`用于查看当前运行的进程，常用组合：

- `ps -aux `
- `ps -ef`

**常用参数**：

- `-a`：显示所有进程，包含每个命令的完整路径
- `-u`：显示所有系统程序，包括那些没有终端的程序
- `-x`：显示使用者的名称和起始时间

### top
`top`用于实时查看服务器进程等信息

**常用参数**：

- `-d`：指定两次屏幕刷新之间的间隔
- `-p`：通过指定监控进程ID来仅仅监控某个进程的状态

### netstat
`netstat`命令用于显示各种网络相关信息，如网络连接，路由表，接口状态。

**常用参数**：

- `-a`：显示所有选项，默认不显示LISTEN相关
- `-t`：仅显示TCP相关的选项
- `-u`：仅显示UDP相关的选项
- `-l`：仅列出有在LISTEN的服务状态
- `-p`：显示建立相关链接的程序名

### lsof
`list open file`命令用于列出当前系统打开文件，在linux环境下，任何事物都以文件存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。

**命令格式**：
 `lsof [参数] [文件]`

**命令功能**：
用于查看你进程打开的问价，打开文件的进程，进程打开的端口（TCP、UDP）。找回/恢复删除的文件。

lsof打开的文件可以是：

- 普通文件
- 目录
- 网络文件系统的文件
- 字符或设备文件
- (函数)共享库
- 管道，命名管道
- 符号链接
- 网络文件（例如：NFS file、网络socket，unix域名socket）
- 还有其它类型的文件，等等


**命令参数**：

- `-a`：列出打开文件存在的进程
- `-c <进程名>`：列出指定进程所打开的文件
- `-g`：列出GID号进程详情
- `-d<文件号>`：列出占用该文件号的进程
- `+d<目录>`：列出目录下被打开的文件
- `+D<目录>`：递归列出目录下被打开的文件
- `-n<目录>`：列出使用NFS的文件
- `-i<条件>`：列出符合条件的进程。（4、6、协议、:端口、 @ip ）
- `-p<进程号>`：列出指定进程号所打开的文件
- `-u`：列出UID号进程详情
- `-h`：显示帮助信息
- `-v`：显示版本信息


# 读取文件每一行并输出
- [读取文件每一行并输出](https://www.cnblogs.com/iloveyoucc/archive/2012/07/10/2585529.html)


# vim使用
## 查找
在normal模式下按下`/`即可进入查找模式，输入要查找的字符串并按下回车，vim会跳到第一个匹配的位置，按`n`查找写一个，`N`查找上一个。

Vim查找支持正则表达式，例如`/vim$`匹配行尾的`"vim"`。 需要查找特殊字符需要转义，例如`/vim\$`匹配`"vim$"`。

> 注意查找回车应当用`\n`，而替换为回车应当用`\r`（相当于<CR>）。


`\c`表示大小写不敏感查找，`\C`表示大小写敏感查找

```
/demo\c
```
将会查找到`"demo"`,`"DEMO"`,`"Demo"`...等
# linux日志

### /var目录
/var所有服务的登陆文件错误或错误文件信息(LOG FILES)都在/var/log下，此外一些数据库如mysql则在/var/lib下，用户未读的邮件默认放在/var/spool/mail

### /var/log
常见系统日志：
- 核心启动日志：/var/log/dmesg
- 系统报错日志：/var/log/messages
- 邮件系统日志：/var/log/maillog
- FTP系统日志：/var/log/xferlog

# sftp命令
### 登录
`sftp -P 22 username@ip`

# telnet命令
telnet命令通常用于远程登录。telnet程序是基于TELNET协议的远程登录客户端程序，是TCP/IP协议族的一员。

在终端使用telnet程序连接到服务器，终端使用者可以在telnet程序中输入命令，这些命令会在服务器上运行，就像直接在服务器的控制台上输入一样。


# ip addr 
linux的ip命令跟ifconfig类似，但是前者功能更强大，并旨在取代后者。

   ![](https://img.linux.net.cn/data/attachment/album/201406/04/003404uy9l1t5zayzllylm.##png)

### 给机器设置ip地址
`sudo ip addr add 192.168.0.193/24 dev wlan0`

### 查看是否生效
`sudo ip addr show wlan0`

### 删除IP地址
`sudo ip addr del 192.168.0.193/24 dev wlan0`

# 后台启动

### 后台启动
linux下一般想让某个程序在后台运行，一般使用`&`

	./test.sh &

### 关闭终端之后继续运行
	nohup ./test.sh &
### 强制结束一个进程
	kill -9 port

# 查看系统信息
### 查看发行版本
	root@a4489e9f9c9e:/usr/src/bg# cat /etc/issue          
	Debian GNU/Linux 9 \n \l
### 查看物理CPU个数
	root@a4489e9f9c9e:/usr/src/bg# cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
	1
### 查看每个物理CPU中core的个数（既核数）
	root@a4489e9f9c9e:/usr/src/bg# cat /proc/cpuinfo| grep "cpu cores"| uniq
	cpu cores	: 1
### 查看逻辑CPU的个数
	root@a4489e9f9c9e:/usr/src/bg# cat /proc/cpuinfo| grep "processor"| wc -l
	1
### 查看CPU信息（型号）
	root@a4489e9f9c9e:/usr/src/bg# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
      1  Intel(R) Core(TM) i5-7500 CPU @ 3.40GHz


### 查看内存
	cat /proc/meminfo
	free -m
### 查看硬盘
	df -h


# 参考
- 鸟哥的linux私房菜基础学习篇
- [awk 入门教程](http://www.ruanyifeng.com/blog/2018/11/awk.html)
- [试试Linux下的ip命令，ifconfig已经过时了](试试Linux下的ip命令，ifconfig已经过时了 "https://linux.cn/article-3144-1.html")
- [Linux的五个查找命令：find,locate,whereis,which,type](Linux的五个查找命令：find,locate,whereis,which,type "http://www.kuqin.com/linux/20091009/70532.html")