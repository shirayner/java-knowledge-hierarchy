[TOC]





# 前言







# 一、bash 手册

通过如下命令可查看制定命令的用法

```bash
man [命令名]
```



# 二、文件管理

## 1.文件

### 1.1 创建

```shell
 touch [OPTION]... FILE...
```

命令解读：

> - change file timestamps
> - 创建指定文件
> - 当文件存在时，更新文件修改时间



### 1.2 复制

```shell
cp [OPTION]... [-T] SOURCE DEST
cp [OPTION]... SOURCE... DIRECTORY
cp [OPTION]... -t DIRECTORY SOURCE...
```

命令解读：

> - copy files and directories
> - Copy SOURCE to DEST, or multiple SOURCE(s) to DIRECTORY.





（1）复制单个文件

当source和destination参数都是文件名时，cp命令将源文件复制成一个新文件，并且以destination命名。

```shell
cp test.txt  test.txt.bak
```



（2）文件覆盖

如果目标文件已经存在，cp命令可能并不会提醒这一点。最好是加上-i选项，强制shell询问是否需要覆盖已有文件。如果不回答y，文件复制将不会继续。

```shell
cp  -i  test.txt  test.txt.bak
```



（3）复制多个文件

若存在dir2不存在，则会创建一个dir1的副本dir2。若dir2存在，则会将dir1拷贝到dir2中。

```shell
cp -R dir1 dir2
```



### 1.3 移动和重命名

```shell
mv [OPTION]... [-T] SOURCE DEST
mv [OPTION]... SOURCE... DIRECTORY
mv [OPTION]... -t DIRECTORY SOURCE...
```

命令解读：

> -  Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.





### 1.4 删除

```shell
rm [OPTION]... FILE...
```

命令解读：

> -  move (rename) files
> - rm removes each specified file.  By default, it does not remove directories



| options              | meanings                                                     |
| :------------------- | ------------------------------------------------------------ |
| -f, --force          | ignore nonexistent files and arguments, never prompt         |
| -i                   | prompt before every removal                                  |
| -I                   | prompt  once before removing more than three files, or when removing recursively; <br/>less intrusive than -i, while still giving protection against  most mistakes |
| --interactive[=WHEN] | prompt according to WHEN: never, once (-I), or always (-i); without WHEN, prompt always |
| -r, -R, --recursive  | remove directories and their contents recursively            |
| -d, --dir            | remove empty directories                                     |



常用命令：

```shell
rm -Ir file
rm -rf file
```



### 1.5 查看

#### 1.5.1 查看文件类型

```shell
file   FILE...
```



命令解读：

> - determine file type



#### 1.5.2 查看整个文件

##### （1）cat

```shell
 cat [OPTION]... [FILE]...
```

命令解读：

> - Concatenate FILE(s), or standard input, to standard output.



| option                | meaning                                    |
| --------------------- | ------------------------------------------ |
| -b, --number-nonblank | number nonempty output lines, overrides -n |
| -n, --number          | number all output lines                    |
| -T, --show-tabs       | display TAB characters as ^I               |



##### （2）more

```shell
more [options] file [...]
```

命令解读：

> - 分页查看文件内容，支持向后翻页



##### （3）less

```shell
 less [options] file [...]
```

命令解读:

>- more的增强，支持前后翻页、高级搜索



#### 1.5.3 查看部分文件

##### （1）tail

```shell
tail [OPTION]... [FILE]...
```

命令解读：

> - output the last part of files
> - Print  the last 10 lines of each FILE to standard output.  With more than one FILE, precede each with a header giving the file name.  With no FILE, or when FILE is -, read standard input.





| option        | meaning                                                      |
| ------------- | ------------------------------------------------------------ |
| -n, --lines=K | output the last K lines, instead of the last 10; or use -n +K to output starting with the Kth |



##### （2）head          

```shell
 head [OPTION]... [FILE]...
```

命令解读：

> - output the first part of files





| option           | meaning                                                      |
| ---------------- | ------------------------------------------------------------ |
| -n, --lines=[-]K | print the first K lines instead of the first 10;<br/> with the leading '-', print all but the last K lines of each file |





      ### 1.6  查找文件
    
      #### 1.6.1 一般查找 find

```shell
find PATH -name FILENAME  # 在 PATH 下查找名为 FILENAME 的文件
```



example:

```shell
[root@localhost ~]# find / -name httpd.conf   # 在根目录查找 httpd.conf
[root@localhost ~]# find / -name *.conf   # 在根目录查找后缀名为 .conf 的文件
```



#### 1.6.2 数据库查找 locate

> 与find不同，locate命令依赖于一个数据库文件，Linux系统默认每天会检索一下系统中的所有文件，然后将检索到的文件记录到数据库中。在运行locate命令的时候可直接到数据库中查找记录并打印到屏幕上，所以使用locate命令要比find命令反馈更为迅速。在执行这个命令之前一般需要执行updatedb命令（这不是必须的，因为系统每天会自动检索并更新数据库信息，但是有时候会因为文件发生了变化而系统还没有再次更新而无法找到实际上确实存在的文件。所以有时需要主动运行该命令，以创建最新的文件列表数据库），以及时更新数据库记录。下面是使用locate命令来查找httpd.conf文件：



```shell
[root@localhost ~]# updatedb 
[root@localhost ~]# locate httpd.conf
```





#### 1.6.3 查找执行文件：which/whereis

which用于从系统的PATH变量所定义的目录中查找可执行文件的绝对路径:

```shell
which passwd
```



使用whereis也能查到其路径，但是和which不同的是，它不但能找出其二进制文件，还能找出相关的man文件：

```shell
whereis passwd
```











## 2.目录

### 2.1 cd-切换目录

cd命令是[change directory]的简写，方便用户切换到不同的目录。

``` shell
cd [directory]  # 目录切换
pwd # 显示当前目录
```



目录有以下两种形式：

- 绝对路径  ：以 根路径 / 开始

- 相对路径

| directory | 描述           |
| --------- | -------------- |
| .         | 当前目录       |
| ..        | 上级目录       |
| ~         | home目录       |
| -         | 上一次所在目录 |





### 2.2 目录列表

```
ls [OPTION]... [FILE]...
```

命令解读：

>- list directory contents





| option          | 描述                                            |
| --------------- | ----------------------------------------------- |
| -l              | use a long listing format                       |
| -a, --all       | do not ignore entries starting with .           |
| -d, --directory | list directories themselves, not their contents |
| -i, --inode     | print the index number of each file             |



### 2.3 创建

```shell
 mkdir [OPTION]... DIRECTORY...
```

命令解读：

> -  make directories
> - Create the DIRECTORY(ies), if they do not already exist



| option        | meanning                                                |
| ------------- | ------------------------------------------------------- |
| -p, --parents | no error if existing, make parent directories as needed |



### 2.4 删除

```shell
 rmdir [OPTION]... DIRECTORY...
```

命令解读：

> - remove empty directories
> - Remove the DIRECTORY(ies), if they are empty.

​             

| option        | meanning                                                     |
| ------------- | ------------------------------------------------------------ |
| -p, --parents | remove DIRECTORY and its ancestors;<br/>e.g., 'rmdir -p a/b/c' is similar to 'rmdir a/b/c a/b a' |



`rmdir`只支持删除空目录，若想删除不为空的目录，则可是使用 `rm`命令——**linux中一切皆文件**

```shell
rm -r dir1
```





# 三、系统状态

## 1.监测进程

### 1.1 探查进程

#### 1.1.1 ps

```shell
 ps [options]
```



命令解读：

> -  report a snapshot of the current processes



具体可选参数请查看 bash手册

常见用法：

```shell
ps  -ef     # 查看系统上运行的所有进程
```



#### 1.1.2 top

实时显示系统上运行的进程信息

```shell
top 
```





### 1.2 结束进程

#### 1.2.1 kill

```shell
kill [OPTION]... pid...
```

命令解读：

> - terminate a process by pid



| option              | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| -s, --signal signal | Specify the signal to send.  The signal may be given as a signal name or number. |
| -l, --list [signal] | Print a list of signal names, or convert signal given as argument to a name. |
| -L, --table         | Similar to -l, but will print signal names and their corresponding numbers. |



示例：

```shell
kill  97878
kill -s HUP 97878
```



#### 1.2.2 killall

```shell
killall [OPTION]... name...
```



命令解读：

> - kill processes by name



示例：

```
killall  http*
```



## 2.监测磁盘

### 2.1 挂载存储媒体

> - Linux文件系统将所有的磁盘都并入一个虚拟目录下。在使用新的存储媒体之前，需要把它放到虚拟目录下。这项工作称为挂载（mounting）。
> - Linux上用来挂载媒体的命令叫作mount。默认情况下，mount命令会输出当前系统上挂载的设备列表。



```shell
mount -t type device directory
```



命令解读：

> - type参数指定了磁盘被格式化的文件系统类型：
>     - vfat：Windows长文件系统。
>     - ntfs：WindowsNT、XP、Vista以及Windows7中广泛使用的高级文件系统。
>     - iso9660：标准CD-ROM文件系统。
> - 后面两个参数定义了该存储设备的设备文件的位置以及挂载点在虚拟目录中的位置。



例如，手动将U盘 `/dev/sdb1` 挂载到 `/media/disk`，可用下面的命令：

```shell
mount -t vfat  /dev/sdb1   /media/disk
```



### 2.2 df

> df 命令可以让你很方便地查看所有已挂载磁盘的使用情况。

```shell
df [OPTION]... [FILE]...
```

命令解读：

> - report file system disk space usage



| option               | 描述                                                    |
| -------------------- | ------------------------------------------------------- |
| -h, --human-readable | print sizes in human readable format (e.g., 1K 234M 2G) |
|                      |                                                         |
|                      |                                                         |



常见用法:

```shell
df  -h
```



### 2.3 du

> du命令可以显示某个特定目录（默认情况下是当前目录）的磁盘使用情况。这一方法可用来快速判断系统上某个目录下是不是有超大文件。



```shell
du [OPTION]... [FILE]...
```

命令解读：

> - estimate file space usage
> -  Summarize disk usage of each FILE, recursively for directories.



| option               | 描述                                                    |
| -------------------- | ------------------------------------------------------- |
| -c, --total          | produce a grand total                                   |
| -h, --human-readable | print sizes in human readable format (e.g., 1K 234M 2G) |
| -s, --summarize      | display only a total for each argument                  |
|                      |                                                         |



# 四、处理数据文件

## 1.排序数据

默认情况下，sort命令按照会话指定的默认语言的排序规则对文本文件中的数据行排序。

```shell
sort [OPTION]... [FILE]...
sort [OPTION]... --files0-from=F
```

命令解读：

> - sort lines of text files
> -  Write sorted concatenation of all FILE(s) to standard output.



| option                    | 描述                                             |
| ------------------------- | ------------------------------------------------ |
| -n, --numeric-sort        | compare according to string numerical value      |
| -r, --reverse             | reverse the result of comparisons                |
| -k, --key=KEYDEF          | sort via a key; KEYDEF gives location and type   |
| -t, --field-separator=SEP | use SEP instead of non-blank to blank transition |





示例：

```shell
du -sh * | sort -nr
```



## 2.搜索数据

```shell
grep [OPTIONS]   PATTERN    [FILE...]
grep [OPTIONS]   [-e PATTERN | -f FILE] [FILE...]
```

命令解读：

> - print lines matching a pattern



| option                       | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -e PATTERN, --regexp=PATTERN | Use  PATTERN  as the pattern.  This can be used to specify multiple search patterns, or to protect a pattern beginning with a hyphen (-).  (-e is specified by POSIX.) |
| -f FILE, --file=FILE         | Obtain patterns from FILE, one per line.  The  empty  file  contains  zero patterns,  and therefore matches nothing.  (-f is specified by POSIX.) |
| -c, --count                  | Suppress normal output; instead print a count of matching lines for each input file.  With  the  -v, --invert-match option (see below), count non-matching lines.  (-c is specified by POSIX.) |
|                              |                                                              |



示例：

```shell
grep -v t file1

grep -n t file1

grep -e t -e f file1  # 指定多个匹配模式

grep [tf] file1       # 使用正则表达式

```



## 3.压缩数据

### 3.1 gzip

> gzip, gunzip, zcat - compress or expand files

```shell
gzip   [ -acdfhlLnNrtvV19 ] [-S suffix] [ name ...  ]
gunzip [ -acfhlLnNrtvV ]    [-S suffix] [ name ...  ]
zcat   [ -fhLV ]                        [ name ...  ]
```

> 首先要弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件。
>
> 为什么要区分这两个概念呢？这源于Linux中很多压缩程序**只能针对一个文件进行压缩**，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）。



| option                       | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -d --decompress --uncompress | Decompress.                                                  |
| -q --quiet                   | Suppress all warnings.                                       |
| -r --recursive               | Travel the directory structure recursively. If any of the file names specified on the command line are directories, gzip will descend into the directory and compress all the files it finds there (or decompress them in the case of gunzip ). |
|                              |                                                              |



示例：

压缩：

gzip不支持将多个文件压缩到一个文件中，因此可配合使用 tar

```shell
gzip -r * 
```



解压：

```shell
gzip -d file.txt.gz
gunzip file.txt.gz
```





## 4.归档数据

```shell
 tar [OPTION...] [FILE]...
```

命令解读：

> - GNU  `tar' saves many files together into a single tape or disk archive, and can restore individual files from the archive.



| option               | 描述                                         |
| -------------------- | -------------------------------------------- |
| -c, --create         | create a new archive                         |
| -r, --append         | append files to the end of an archive        |
| -t, --list           | list the contents of an archive              |
| -x, --extract, --get | extract files from an archive                |
| -v, --verbose        | verbosely list files processed               |
| -f, --file=ARCHIVE   | use archive file or device ARCHIVE           |
| -u, --update         | only append files newer than copy in archive |
| -z, --gzip           | filter the archive through gzip              |




​              

> -f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。    

示例：

```shell
打包： tar -cvf [目标文件名].tar [原文件名/目录名]
解包： tar -xvf [原文件名].tar
打包并压缩： tar -zcvf [目标文件名].tar.gz [原文件名/目录名] 
解压并解包： tar -zxvf [原文件名].tar.gz              # 注：z代表用gzip算法来压缩/解压。
```



​              

# 四、其他命令

## 1.命令替换

命令替换允许你将shell命令的输出赋给变量。



有两种方法可以将命令输出赋给变量：

-  反引号符(`)
- $()  格式



```shell
testing=$(date)
testing1=`date`
echo "The date and time are: " testing1
```





## 2.重定向输入输出

- 有些时候你想要保存某个命令的输出，而不仅仅只是让它显示在显示器上。

- bashshell提供了几个操作符，可以将命令的输出重定向到另一个位置（比如文件）。

- 重定向可以用于输入，也可以用于输出，可以将文件重定向到命令输入。



### 2.1 重定向输出

最基本的重定向将命令的输出发送到一个文件中：

```shell
command > outputfile   #  将命令输出重定向到outputfile, 会覆盖文件
```



example:

```shell
[ray@localhost dev]$ date  > file1          # 将命令输出重定向到file1
[ray@localhost dev]$ ls -l file1
-rw-rw-r--. 1 ray ray 29 Jan 24 15:21 file1
[ray@localhost dev]$ cat file1
Thu Jan 24 15:21:14 CST 2019
```



有时，你可能并不想覆盖文件原有内容，而是想要将命令的输出**追加**到已有文件中：

```shell
command >>  outputfile   # 将命令输出以追加的方式重定向到 outputfile
```





### 2.2 重定向输入

输入重定向将文件的内容重定向到命令

```
command  <  inputfile
```



## 3.管道

有时需要将一个命令的输出作为另一个命令的输入。这可以用重定向来实现，只是有些笨拙。

```
rpm -qa > rpm.list
sort < rpm.list
```

上述过程可以管道来进行简化。



管道被放在命令之间，将一个命令的输出重定向到另一个命令中：

```
command1 | command2
```

> - 不要以为由管道串起的两个命令会依次执行。Linux系统实际上会同时运行这两个命令，在系统内部将它们连接起来。
> - **在第一个命令产生输出的同时，输出会被立即送给第二个命令**。数据传输不会用到任何中间文件或缓冲区。



exmaple:

```shell
rpm -qa  | sort
```











