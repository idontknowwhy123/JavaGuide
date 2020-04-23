1.chmod +755和chmod +777 各是什么意思<br>
755 代表用户对该文件拥有读，写，执行的权限，同组其他人员拥有执行和读的权限，没有写的权限，其他用户的权限和同组人员权限一样。。。<br>
777代表，user,group ,others ,都有读写和可执行权限。。<br>
https://www.cnblogs.com/z-x-y/p/9287694.html
给文件夹赋权
chmod 777 -R /log

2.删除文件使用实例： <br>
rm -f  python.py<br>
将会强制删除python.py这个文件 <br>

删除目录使用实例： <br>
rm -rf  /home/song/wahaha <br>
将会删除/home/song/wahaha目录以及其下所有文件、文件夹 <br>

3.tail -f .log实时查看日志
  tail -n 500查看日志最后500行
  Head -n 500查看日志前500行
  
4.curl url(curl命令可以通过restClient生成)查看某个url请求的结果

5.括号转义\(123\)

6.pwd(查看当前目录)

7.touch 123.txt(创建文件)

8.mkdir 123(创建文件夹)

9.同平台拷贝(cp)
  跨平台拷贝(scp)
  
10.想要回退到一个特定的父目录
cd../../../../
回到上一次目录：cd - (可以穿越多层)

11.Linux 下面解压.tar.gz 和.gz文件解压的方式
两种解压方式
1.tar.gz 使用tar命令进行解压

 tar -zxvf java.tar.gz

解压到指定的文件夹
    tar -zxvf java.tar.gz  -C /usr/java
2.gz文件的解压 gzip 命令

  gzip -d java.gz
  
3.tar –xvf file.tar //解压 tar包

4.unzip file.zip //解压zip

如果出现这个提示：
-bash: zip: command not found    不能执行ZIP压缩，是因为没有安装ZIP，
运行下这条安装命令即可  yum install zip

12修改文件名字mv name1 name2

13堆栈打印
1.ps -ef|grep "进程id"
2.jstack -l pid打印堆栈信息(kill-3 pid也行)

14查看某个线程的堆栈信息找到问题代码(CPU彪高)
1.top找到CPU消耗最高的进程ID
2.top -Hp p 进程id找出cpu消耗最高的线程id
3.jstack -l 进程id>123.txt将线程堆栈信息输出到某个文件中
4.由于刚刚线程id是10进制，而线程堆栈信息是16进制，所以需要将其转化为16进制
  printf"%x\n" 十进制线程id
5.通过刚刚转换过的16进制线程id去堆栈文件中去找对应的线程堆栈

https://blog.csdn.net/u010271462/article/details/70171553

14查看某个容器进程的线程数
1.找出容器进程
docker ps|grep "service"找出容器Id
docker top CONTAINERID找出容器的进程ID
top -Hp 容器进程id、

15Linux -->在目录内创建文件、显示文件以及拷贝文件到一个目录都需要什么权限？
https://blog.csdn.net/z517602658/article/details/62076193
1.在目录内创建文件我们需要用户的写权限。
2.加上读权限后便可以显示刚刚创建的文件了
3.只有可执行和读才可拷贝文件
4.进入一个目录需要执行权限

16、查看某个目录所在位置
 which 目录 whereis 目录
 
17.打印堆栈信息
1.找到java路径 whereis java
2.找到服务进程号 ps -ef|grep "服务"
3.通过进程号打印堆栈 在java/bin目录下执行./jstack -l pid

18.编译并执行某个类
   1.找到java安装路径 whereis java 假如是home/ct/java
   2.执行编译：/home/ct/java/bin/javac test.java -Djava.ext.dirs=/home/ct/java/dependency/(有时需要加上-encoding UTF-8)
   3.执行java的main方法：/home/ct/java/bin/java -cp .:1.jar:2.jar:3.jar test(需要依赖1.jar,2.jar,3.jar)
   
 19.查找某个文件
   find / -name test.java

20.查看某个端口占用情况
  netstat -anlp|grep "pid"

