使用docker ps 命令即可列出运行中的容器，docker ps -a命令即可列出所有容器（包括未运行的）
执行该命令后，会出现如下7列表格

1  CONTAINER_ID              表示容器ID

2  IMAGE                     表示镜像名称

3 COMMAND             表示启动容器时运行的命令

4  CREATED                表示容器的创建时间

5 STATUS                   表示容器运行的状态。UP表示运行中，EXITED表示已停止
                          (EXITED为1的话一般是应用程序出问题，EXITED为137的话一般是OOM)

6 PORTS                     表示容器对外的端口号

7 NAMES                    表示容器名称，该名称默认由Docker自动生成，也可使用docker run 命令的 -name 选项自行指定

docker ps -a|grep (某个容器内的2微服务)：查找某个微服务对应的docker容器信息

删除镜像
docker rm IMAGE ID

查看镜像
docker images

查看容器 
docker ps -a<br>
docker ps|grep itmmps

启动/停止容器
docker start/stop docker_id

重启容器
docker restart docker_id

进入容器 
docker exec -it docker_name /bin/sh
docker exec -it docker_id /bin/sh
docker exec -ti docker_id /bin/sh

docker容器中的文件拷贝到本地服务器上<br>
1.查询容器id：docker ps|grep itmmps<br>
2.docker cp docker_id:/home/zenap/itmmps/itm-mps.jar /home/ubuntu/(.当前目录)<br>
3.scp -r ubuntu@10.74.165.77:/home/ubuntu /home/ubuntu/(.当前目录)<br>
4.通过filezilla等工具拷贝到本地服务上<br>

本地服务器的文件拷贝到docker容器中<br>
1.通过filezilla等工具拷贝到远程服务器上<br>
2.将服务器上的文件拷贝到小网ip，eg：scp -r /home/ubuntu/itm-mps.jar ubuntu@192.167.0.7:/home/ubuntu<br>
3.查询容器id：docker ps|grep itmmps<br>
4.docker cp /home/ubuntu/itm-mps.jar docker_id:/home/zenap/itmmps<br>

对容器所有请求读写抓包方法（用于排查问题）

ps -ef|grep ssm-server<br>
root      94373  94287 17 Feb21 ?        02:42:39 /openj9jdk/bin/java -Xms200m -Xm<br>
sudo strace -p 94373 -f -e trace=network -s 1000
