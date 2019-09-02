1.chmod +755和chmod +777 各是什么意思<br>
755 代表用户对该文件拥有读，写，执行的权限，同组其他人员拥有执行和读的权限，没有写的权限，其他用户的权限和同组人员权限一样。。。<br>
777代表，user,group ,others ,都有读写和可执行权限。。<br>

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
