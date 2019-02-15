使用docker ps 命令即可列出运行中的容器，docker ps -a命令即可列出所有容器（包括未运行的）
执行该命令后，会出现如下7列表格

1  CONTAINER_ID      表示容器ID

2  IMAGE                     表示镜像名称

3 COMMAND             表示启动容器时运行的命令

4  CREATED                表示容器的创建时间

5 STATUS                   表示容器运行的状态。UP表示运行中，EXITED表示已停止

6 PORTS                     表示容器对外的端口号

7 NAMES                    表示容器名称，该名称默认由Docker自动生成，也可使用docker run 命令的 -name 选项自行指定
