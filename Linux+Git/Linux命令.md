

### 练习

#### 目录管理练习

1. 打开用户主目录
2. 显示当前路径
3. 创建文件夹
4. 递归创建文件夹
5. 打开文件夹
6. 创建文件
7. 创建文件硬链接与软链接
8. 编辑文件
9. 编辑文件时显示行数
10. 保存文件并退出
11. 查看文件
12. 向文件追加文本
13. 修改文件权限
14. 统计文件行数、单词数、字符数
15. 切回根目录
16. 查找文件
17. 移动文件、移动目录、文件重命名
18. 复制文件、复制目录
19. 删除目录、递归删除目录
20. 删除用户、删除用户及用户目录



#### 用户管理练习

1. 创建新用户
2. 创建新用户并指定用户目录
3. 创建新用户并指定用户目录并设置密码
4. 切换用户
5. 查看用户相关文件
6. 修改用户属组
7. 修改用户密码、root修改用户密码
8. 删除用户



#### 系统管理

1. 查看硬盘状态
2. 查看所有进程、查看指定进程
3. 关闭进程
4. 查看防火墙状态
5. 打开防火墙、查看开放端口、关闭防火墙
6. 

- 基本命令

```shell
ls
ls -a
ls -l
ll


echo

cat

head
head -n
tail 
tail -n
cat -n


date

cal

who


vim

mkdir
mv

rm
rm -r
rm -f

cp
cp -r

chown

/etc/passwd
/etc/shadow
/etc/group

lsattr 
chattr +a/i

：nohl	//取消高亮

whereis
which
```





#### Linux命令

#### 文件管理

```shell
#创建目录
mkdir [directory] [directory2] ...
mkdir -p directory/directory2/...

#删除目录
```

### 用户管理

#### 配置文件

```shell
#用户信息文件
/etc/passwd
#密码文件
/etc/shadow
#用户组文件
/etc/group
#用户组密码文件
/etc/gshadow
#用户配置文件
/etc/login.defs
/etc/default/useradd
#新用户信息文件
/etc/skel
#登录信息
/etc/motd
/etc/issue
```

##### /etc/passwd

![image-20210816173426700](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210816173426700.png)

![image-20210816173502764](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210816173502764.png)

![img](https://img-blog.csdn.net/20180905090606798?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwOTA0OTg1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### /etc/shadow

![image-20210817082936340](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210817082936340.png)

![img](https://img-blog.csdn.net/20180905091800466?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwOTA0OTg1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### /etc/group

![image-20210817083050294](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210817083050294.png)

![img](https://img-blog.csdn.net/20180905092024217?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwOTA0OTg1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



##### 使用手动的方式添加用户

1. 在/etc/passwd	/etc/shadow	/etc/group中各添加一行记录

```shell
echo 'newuser:x:5000:5000:newuser:/home/newuser:/bin/bash' >> /etc/passwd
echo 'user:!!:::::::' >>/etc/passwd
echo 'mygroup:x:5000:newuser' >> /etc/group
```

2. 创建用户宿主目录

```shell
mkdir /home/newuser
```

3. 在用户宿主目录中设置默认的配置文件

```shell
#复制/etc/skel/目录下的内容copy到/home/newuser/下
cp -r /etc/skel/ /home/newuser/
```

4. 修改home目录的权限、属主、属组
5. 生成密码并放入user的密码字段中

#### 用户类型

- 超级用户（root，UID=0）
- 普通用户（UID 500-60000）
- 伪用户（UID 1-499）

##### 超级用户

- 指超级管理员，超级用户只有一个，具有全部权限

##### 普通用户

- 指由超级管理员创建的用户，其权限与属组相关

##### 伪用户

- 伪用户与系统进程相关
- 伪用户通常无法登陆系统
- 可以没有宿主目录

#### 用户组

- 每个用户至少属于一个用户组
- 每个用户组包括多个用户
- 同一用户组的用户享有该组的全部权限

#### 添加用户

两个用户创建命令之间的区别

- adduser： 会自动为创建的用户指定主目录、系统shell版本，会在创建时输入用户密码。

- useradd：需要使用参数选项指定上述基本设置，如果不使用任何参数，则创建的用户无密码、无主目录、没有指定shell版本。

```shell
#创建user
sudo  useradd  tt
#为新用户添加密码
sudo passwd tt


为用户指定参数的useradd命令：

常用命令行选项：
（1） -d：           指定用户的主目录

（2） -m：          如果存在不再创建，但是此目录并不属于新创建用户；如果主目录不存在，则强制创建； -m和-d一块使用。

（3） -s：           指定用户登录时的shell版本

（4） -M：           不创建主目录
```

#### 删除用户

```shell
userdel user
#删除用户和主目录
userdel -r user
```



## 扩展

```shell
为文件添加i的权限命令是
#增加i的权限
chattr +i fileName
#去除i的权限
chattr -i fileName
#增加a的权限，内容可以追加，不能删除
chattr +a fileName
#除去a的权限
chattr -a fileName
```



### 文件管理

#### 文件属性

##### 使用shown修改文件的权限

```shell
#修改文件的属主与属组
chown pikachu:pikachu -R file
#修改文件的属主
chown pikachu -R file
#修改文件的属组
chown :pikachu -R file
```

## 链接文件

1.**Linux文件管理特性**

文件都有文件名与数据，在Linux上被分成两个部分：
**用户数据 (user data) 与元数据 (metadata)**。

**用户数据，即文件数据块** (data block)，数据块是**记录文件真实内容**的地方，
**元数据则是文件的附加属性**，如文件大小、创建时间、所有者等信息。

在Linux中**元数据中的inode 号**（inode 是文件元数据一部分但其并不包含文件名，inode 号即索引节点号）**才是文件的唯一标识而非文件名**。文件名仅是为了方便人的记忆和使用，系统或程序通过 inode 号找 正确文件数据块。

2、 **为什么会有链接文件**

**为解决文件的共享使用**，Linux 系统引入了链接，链接为Linux 系统解决了文件的共享使用，还**带来了隐藏文件路径、增加权限安全及节省存储等好处**。创建链接文件可以给文件和目录创建替代名或别名。

**分类：**

**硬链接** (hard link)
**软链接**（又称符号链接，即 soft link 或 symbolic link）

ln source_file target_file
建立硬链接

> **ln [-s或P] source_file target_file**
> -s：建立软连接
> -P：建立硬链接（注意是大写）

## **硬链接和软连接的区别**

**1.硬链接**

若**一个 inode 号对应多个文件名**，则称这些文件为硬链接。换言之 ，**硬链接就是同一个文件使用了多个别名**。只能在同一个文件系统来链接文件，源文件和目标文件同一个文件，当有多个硬链接文件时，删除其中一个则是删除了一个文件名而已，硬链接数少1个。

由于硬链接是有着相同 inode 号仅文件名不同的文件，因此硬链接存在以下几点特性：

1）文件有相同的 inode 及 data block；
2）只能对已存在的文件进行创建；
3）不能交叉文件系统进行硬链接的创建；
4）**不能对目录进行创建**，只可对文件创建；
5）删除一个硬链接文件并不影响其他有相同 inode 号的文件。

![这里写图片描述](https://img-blog.csdn.net/20180325205805531?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2p1bmp1bmJhMjY4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**2.软链接**

若**文件用户数据块中存放的内容是另一文件的路径名的指向**，则该文件就是软连接。软链接就是一个普通文件，只是数据块内容有点特殊。软链接有着自己的 inode 号以及用户数据块。

可在不同的文件系统来链接文件，**源文件和目标文件是不同文件**，有不同的大小，是2个文件， **目标文件的内容是源文件的inode号指向源文件，像windows中的快捷方式一样**。

建立了软连接后，**软连接文件的大小是指向的目标文件的文件名的大小**；

1）软链接与硬链接不同，软链接创建与使用没有类似硬链接的诸限制 ；
2）软链接有自己的文件属性及权限等；
3）**可对不存在的文件或目录创建软链接**；
4）软链接**可交叉文件系统**；
5）**软链接可对文件或目录创建**；
6）创建软链接时，**链接计数 i_nlink 不会增加**；
7）删除软链接并不影响被指向的文件，但若被指向的原文件被删除，则相关软连接被称为**死链接**（即 dangling link，若被 指向路径文件被重新创建，死链接可**恢复**为正常的软链接）。

## 拷贝移动文件

**cp**
**拷贝文件**
cp [-i] source_file destination_file
cp [-i] source_file(s) destination_directory
-i选项作用：当目标文件存在，会询问是否覆盖，没有-i选项则不询问直接覆盖

> cp wj.txt wj1.txt ： 拷贝wj到wj1

**拷贝目录**
**cp -r source_directory(s) destination_directory(s)**

> cp -r dir1 dir2 :dir1拷贝到dir2目录下
> cp -r dir3 dir4 dir5 :dir3,dir4拷贝到dir5目录下

**mv**
**移动文件目录或重命名文件目录**
mv [-i] source_file target_file 重命名源文件为目标文件
mv [-i] source_file target_directory 移动文件到目标目录

> mv wj.txt wj1.txt :在本目录下将wj重命名为wj1
> mv wj1.txt dir1：将本目录下的wj1移动到dir1目录下（若dir1不存在，则是对源文件进行改名：如 mv userinfo dir6）

## 文件目录权限

Linux系统中的每个文件和目录都有**访问许可权限**，用他来确定谁能通过何种方式对文件和目录进行访问和操作。

**权限的分类:**

**r 读权限**：可以打开文件、目录读取查看；
**w 写权限**：对文件、目录可以编写更改；
**x 执行权限**：对文件可执行(可执行文件)、对目录可查找该目录下的内容；

- 没有权限如 ls -l
- -rwxr-xr-x

**文件目录权限定义:**
![这里写图片描述](https://img-blog.csdn.net/20180326091321898?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2p1bmp1bmJhMjY4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**权限所属对象:**

**1.拥有者**
生成文件或目录时登陆的当前人，权限最高，用u表示。

**2.同组人**
系统管理员分配的同组的一个或几个人，用g表示。

**3.其他人**
除拥有着，同组人以外的人，用o表示。

**4.所有人**
包括拥有着、同组人及其他人，用a表示。

------

## 修改文件目录文件

**chmod**
**修改文件目录的访问权限**，修改权限的前提条件是在修改权限时，要注意自己是文件。

**1.使用字母表示权限**
![这里写图片描述](https://img-blog.csdn.net/20180326092653540?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2p1bmp1bmJhMjY4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
chmod u=r,g+w,o-x wj.txt ：可一次设多种权限1
```

**2.使用数字表示权限**
使用八进制数字来表示权限

r w x
0 0 0 无权限
1 1 1 有权限
![这里写图片描述](https://img-blog.csdn.net/20180326092744347?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2p1bmp1bmJhMjY4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
chmod 555 wj.txt ：三种用户都是读执行权限，即
-r-xr-xr-x 1 wj wj 0 Sep 19 20:28 wj.txt
ubuntu默认建立文件时 赋值664 即读写读写读123
```

**chown**
**更改某个文件或目录的属主和属组，可用于授权；**
例如root用户把自己的一个文件拷贝给用户xu，为了让用户xu能够存取这个文件，root用户应该把这个文件的属主设为xu，否则，用户xu无法存取这个文件。

```
chown [选项] 用户或组 文件
sudo useradd wj1 ：添加用户wj1
sudo passwd wj1 ：给wj1用户设置密码
sudo chown wj1 wj.txt :更改wj文件的属主
（sudo命令以系统管理者的身份执行指令）12345
```

**chown将指定文件的拥有者改为指定的用户或组。**用户能是用户名或用户ID。组能是组名或组ID。

文件是以空格分开的要改动权限的文件列表，支持通配符。

-R 递归式地改动指定目录及其下的所有子目录和文件的拥有者。
-v 显示chown命令所做的工作。

chownwangshiyan.cchownwangshiyan.c chown -R wang users /his

**chgrp**
**改动文件或目录所属的组。**

```
chgrp [选项] group filename
sudo groupadd wj1 ：添加组
sudo chgrp wj1 wj.txt ：指定文件所属的用户组123
```

**该命令改动指定指定文件所属的用户组。**其中group能是用户组ID，也能是/etc/group文件中用户组的组名。文件名是以空格分开的要改动属组的文件列表，支持通配符。如果用户不是该文件的属主或终极用户，则不能改动该文件的组。
\- R 递归式地改动指定目录及其下的所有子目录和文件的属组。
$ chgrp –R book /opt/local /book。
改动/opt/local /book/及其子目录下的所有文件的属组为book。

## 查找文件

**find**
**file path expression [action] 查找文件和目录**

**前提条件**：要对被查找的目录及其所有子目录**有读权限**才能查找。
**查找选项**：通过文件属性来查找

-name 按文件名
-user 按用户(文件属主)
-size 按大小
-mtime 按最后一次修改时间
-atime 按最后一次访问时间
-type 按文件类型 f:file d:directory
-perm 按权限

**find / -name b\***
//找根目录下面名字以b开头的所有文件

**find . -mtime 10 -print**
//查找当前目录下最后一次修改时间距离今天之前10天的那一天修改的文件和目录，并显示出来

**find /etc -user 0 -size +400 -print**
//查找根目录下的/etc下的子目录中由用户id=0，创建的文件大小要大于200k的，并把它显出来

**find ~ -perm 777 > ~/holes**
//在用户住目录下查找权限为777的，即拥有者，同组的，和其他人的的权限都具有读写权限的文件和子目录，并且将查找结果都放在用户主目录下的/holes文件中

**find /export/home -type f -atime +365 -exec rm {}\;**
//查找/export/home下的文件，最后一次访问时间是距离今天是大于365天的文件，再将找到的文件执行一个进程，并删除这些文件。\;代表转义，即就代表分号本身。

**Locate**
**速度比find快**
**locate [-d <数据库文件>][–help][–version][keywords]**

**locate指令用于查找符合条件的文件**，它会去保存文件与目录名称的**数据库内查找**合乎范本样式条件的文件或目录。

-d<数据库文件>或–database=<数据库文件> ：设置locate指令使用的数据库。
**locate指令预设的数据库位于/var/lib/slocate目录**里，文件slocate.db，您可使用这个参数另行指定。
–help 在线帮助。
–version 　显示版本信息。

速度快很多，它是**通过inode，文件索引来找**，它会把文件索引维护在一个数据库里面，它在数据库去找；比较麻烦的是**需要更新数据updatedb**。

```
locate -c b* ：查看当前以bj开头的文件个数
locate --help：查看帮助文档
locate -V：查看版本
```

## 过滤与统计

**grep**
**查出包含某些字符串的结果，对文件或输出结果进行过滤**，大小写敏感。

grep [option(s)] string filename
-i 忽略大小写
-v 反向匹配（查出不包含字符串的结果）

```
grep wj /etc/passwd 查看wj下的有关信息1
ls -la | grep -i 'Sep 18' 
将结果过滤，查看当前目录下的最后修改时间
为9月18的文件和目录，忽略大小写123
ls wj*.txt | xargs grep hello 
在前面的命令结果中，将wj开头的和.txt结尾的文件
逐一交给grep过滤出包含hello内容的123
```

**wc**
**统计文件或输出结果**

**wc [option(s)] filename(s)**
-l 统计多少行
-w 统计多少个单词
-c 统计多少个字符

```
who | grep wj | wc -l
统计当前linux系统中以wj登录的用户个数。
who是当前系统中的用户。

grep li /etc/passwd | wc -l
统计用户名里面有li的

wc -w wj.txt ：统计文件里的单词个数 
wc -c wj.txt ：统计文件里的字符数
cal > wj1.txt ：将日历输出到wj1
wc -c wj.txt：统计文件里的字符数
```
