

`进入目录`   ,`打印目录`,`查看当前路径`,`创建文件`,

`删除文件夹`,`删除文件`,`复制文件`,`移动文件/重命名`,

`查看文件内容`,`查看文件最后的内容`,`分页查看内容`,`编辑文件，编辑文件的操作`,

`查看文件中的关键词`,`查找文件`,`压缩tar文件`,`gzip操作`,

`解压zip操作`,`查看命令作用`,`查看当前用户名`,`查看系统`,

`查看磁盘使用情况`,`查看运行进程`,`查看cpu运行情况（扩展）`,`查看空闲的内存区域`,



# 1,	cd     命令：进入目录

```shell
cd /  		# 进入根目录
cd a    	#进入当前目录下的a目录
cd ..  		#返回上级目录
cd   a/b/c  #去当前目录下的a目录下的b目录下的c目录
```

# 2,	ls   命令：输出目录

```shell
ls -l   	#列出当前目录下文件的属性权限等
ls -a    	#列出所有文件，包括隐藏文件
ls -r    	#输出当前目录及子目录下的所有文件
```

# 3,	pwd 命令：查看当前路径

```shell
pwd 		#查看当前所在目录
```

# 4,	mkdir 命令：创建文件夹

```shell
mkdir a 	#在当前目录创建a文件夹
mkdir -p a  #如果当前文件夹没有a文件夹，就创建一个
```

# 5,	rmdir	命令：删除文件夹

```shell
redir a   	#删除空的a目录
redir -p a/b 	#将a目录下b目录删除，如果b目录删除后，a为空目录，也删掉
```

# 6,rm命令：删除文件

```shell
rm 文件名 #删除文件，且系统会询问
rm -f 文件名 #删除文件，系统不询问
rm -i *.txt #删除任何一个.txt文件，都会询问
rm -r test #将test子目录和子目录下的所有文档删除
rm -rf test #将test子目录和子目录所有文档删除，不用确认

```

# 7,cp命令：复制文件

```shell
cp 文件 路径 #将文件复制到指定路径
cp -r 文件夹a 文件夹b #将文件夹复制到指定路径
cp 文件 文件2 #将文件在当前目录复制一份
```



# 8,mv命令：移动文件/重命名

```shell
mv 文件名 文件名2 #重命名
mv 文件名 路径   #移动文件到指定路径
```



# 9,cat命令：查看文件内容

```shell
cat test1.txt #在屏幕上查看文件内容
```

# 10,tail命令：查看文件最后的内容

```shell
tail d.txt   #显示d.txt最后十行
tail -n 1 d.txt #显是最后1行的消息
```

# 11,less命令：分页查看文件内容

```shell
less d.txt #显示d.txt
#使用ctrl+f/ctrl+b  翻页
```



# 12,vim命令：编辑文件

```shell
vim 文件名 #查看并编辑文件


a #进入编辑模式
:wq#保存并退出
:q! #直接退出
/查找的内容 \C#查找大小写敏感
/查找的内容 \c#查找大小写不敏感
:s/foo/bar/g #在当前行把foo全部替换成bar，g意义是全部替换
:%s/foo/bar/g#全局把foo替换成bar

:nohl #取消查找
```

# 13,grep命令：查看文件中关键词

```shell
grep 文件中内容 *txt #查看当前文件夹中，有”文件中内容“的txt文件的那一行
grep -A <显示行数> 文件中内容 *txt #同时显示后面两行
grep -B <显示行数> 文件中内容 *txt #同时显示前几行内容
```

# 14,find命令：查找文件

```shell
find a #查找a文件
find -name a #区分大小写查找
find -iname a#不区分大小写查找
```

# 15,tar命令：tar压缩文件

```shell
tar -cvf tar1.tar t1.txt#创建tar包
tar -tvf tar1.tar #查看对应的tar包
tar -xvf tar1.tar#提取对那个tar包
```



# 16,gzip命令：创建压缩文件

```shell
gzip a.txt#创建gz文件
gzip -d a.txt.gz #提取文件
```

# 17,unzip命令：解压当前文件

```shell
unzip a.txt.zip #解压zip文件
```

# 18,whatis命令：查看命令作用

```shell
whatis pwd #查看pwd命令的作用
```

# 19,who命令：查看当前用户名

```bash
who
```

# 20,uname命令：查看系统

```shell
uname# 查看系统名
uname -a#查看详细的系统信息
```

# 21,df命令：查看磁盘使用情况

```bash
df	#查看磁盘使用情况
df	-a#全部文件系统列表
df -h #方便阅读显示
```

# 22,ps命令：查看运行进程

```shell
ps #查看当前正在运行的程序
```

# 23,top命令：查看cpu运行情况

```shell
#第一行任务队列信息(时间，用户)
#第二行任务进程
#第三行cpu状态信息（用户空间占cpu，内核空间，空闲cpu，io等待cpu）
#第四行内存状态（物理内存总量，使用中，空闲，缓存）
#第五行swap交换分区信息（总量，使用中，空闲，缓冲中）

#进程（id，所有者，优先级，nice值，虚拟内存，等）
top	#查看cpu运行情况
```

![image-20210324110759626](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210324110759626.png)

![image-20210324110830896](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210324110830896.png)

# 24，free命令：查看空闲内存区域

```shell
free #查看空闲区域
free -h#看的舒服些，单位换算
```

