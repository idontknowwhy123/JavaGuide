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
