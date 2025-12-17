# 写在前面
本文很多内容其实都来[runnoob.com](https://www.runoob.com/docker/docker-command-manual.html) ,为了方便查阅，制作此整合,仅作学习用
# Docker对容器基本操作
#### **docker run**
- 含义:docker run 命令用于创建并启动一个新的容器。  
- 语法:  
`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`  
- 常用参数:
```
-d 后台守护进程  
-it 交互式+伪端口  
--name 取名  
-p 端口映射 格式:host_port:container_port  
--rm 在容器停止后删除
-u 指定用户
-v 挂载卷 格式:host_dir:container_dir  
```
#### **docker start/stop/restart**
- 含义:就是字面意思
- 语法:  
`docker start/stop/restart [OPTIONS] CONTAINER [CONTAINER...]`
- 常用参数:
```
-t, --time 停止容器之前等待的秒数，默认是 10 秒。
```
#### **docker kill**
- 含义:立即终止一个或多个正在运行的容器。
- 语法:
`docker kill [OPTIONS] CONTAINER [CONTAINER...]`
- 常用参数:
```
-s, --signal: 发送给容器的信号(默认为 SIGKILL)
- SIGKILL: 强制终止进程（默认信号）。
- SIGTERM: 请求进程终止。
- SIGINT: 发送中断信号，通常表示用户请求终止。
- SIGHUP: 挂起信号，通常表示终端断开。
```
#### **docker rm**
- 含义:删除**停止的**容器
- 语法:
  `docker rm [OPTIONS] CONTAINER [CONTAINER...]`
- 常用参数:
```
-f, --force:强制删除正在运行的容器
```
- 扩展  
`docker container prune`会删除所有停止的容器  
#### **docker create**
- 含义:创建一个新的容器，**但不会启动它。**  
- 语法：  
`docker create [OPTIONS] IMAGE [COMMAND] [ARG...]`  
- 常用参数:
```
--name 取名  
-p 端口映射 格式:host_port:container_port  
--rm 在容器停止后删除
-u 指定用户
-v 挂载卷 格式:host_dir:container_dir  
--entrypoint 覆盖容器的默认入口点
--detach 在后台创建容器。
```
#### **docker exec**
- 含义:命令用于在**运行中**的容器内执行一个新的命令
- 语法:  
`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]` 
- 常用参数:  
```
-d, --detach: 在后台运行命令
--privileged: 给这个命令额外的权限
--user, -u: 以指定用户的身份运行命令
--workdir, -w: 指定命令的工作目录
-it 交互式+伪端口  
```
#### **docker rename**
- 含义:字面意思,重新命名一个容器
- 语法:  
`docker rename <当前容器名称或ID> <新容器名称>`
#### **docker ps**
- 含义:列出 Docker 容器，默认运行中的
  -语法:  
 `docker ps [OPTIONS]`  
 - 常用参数
```
-a, --all: 显示所有容器，包括停止的容器
-q, --quiet: 只显示容器 ID
-s, --size: 显示容器的大小
--filter, -f: 根据条件过滤显示的容器
--format: 格式化输出
-l, --latest: 显示最近创建的一个容器，包括所有状态
-n [number]: 显示最近创建的 n 个容器，包括所有状态

常见过滤器
status: 容器状态（如 running、paused、exited）。
name: 容器名称。
id: 容器 ID。
label: 容器标签。
ancestor: 容器镜像。
```
#### **docker stats**
- 含义:**实时**显示 Docker 容器的资源使用情况  
- 语法:  
`docker stats [OPTIONS] [CONTAINER...]`
- 常用参数:
```
--all , -a :显示所有的容器，包括未运行的
--format :指定返回值的模板文件
--no-stream :展示当前状态就直接退出了，不再实时更新
```
#### **docker inspect**
- 含义:获取 Docker 对象（容器、镜像、卷、网络等）的详细信息,并返回 JSON 格式的详细信息。
- 语法:  
`docker inspect [OPTIONS] NAME|ID [NAME|ID...]`
- 常用参数:  
```
-f, --format: 使用 Go 模板语法格式化输出。
--type: 返回指定类型的对象信息（可选类型：container、image、network、volume）。
```
#### **docker events**
- 含义:用于实时获取 Docker 守护进程生成的事件,允许用户监控 Docker 容器、镜像、网络和卷的各种操作事件  
- 语法:  
  `docker events [OPTIONS]`
- 常用参数:  
```
-f, --filter: 根据提供的条件过滤输出。
--format: 使用 Go 模板格式化输出。
--since: 显示从指定时间开始的事件。
--until: 显示直到指定时间的事件。
```
# Docker对容器内操作
#### **docker logs**
- 含义:获取和查看容器日志输出  
- 语法:  
  `docker logs [OPTIONS] CONTAINER`
- 常用参数:
```
-f, --follow: 跟随日志输出（类似于 tail -f）。
--since: 从指定时间开始显示日志。
-t, --timestamps: 显示日志时间戳。
--tail [number]: 仅显示日志的最后[number]部分
--details: 显示提供给日志的额外详细信息。
--until: 显示直到指定时间的日志。
```
#### **docker top**
- 含义:显示指定容器中运行的命令
- 语法:  
  `docker top [OPTIONS] CONTAINER [ps OPTIONS]`  

#### **docker attach**
- 含义:附加到正在运行的 Docker 容器,允许用户直接与容器交互  
- 语法:
  `docker attach [OPTIONS] CONTAINER`
#### **docker export**
- 含义:将 Docker 容器的文件系统导出为一个 tar 归档文件。**不包括 Docker 镜像的所有层和元数据。**
- 语法:  
`docker export [OPTIONS] CONTAINER`  
- 常用参数:
```
-o, --output: 将输出保存到指定文件，而不是输出到标准输出。
```
#### **docker import**
- 含义:与export相对
- 语法:  
`docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]`
- 常用参数
```
-m, --message: 为导入的镜像添加注释。
```
#### **docker port**
- 含义:显示容器的端口映射信息
- 语法:  
`docker port CONTAINER [PRIVATE_PORT[/PROTO]]`
#### **docker update**
- 含义:更新容器资源限制
- 语法:  
`docker update [OPTIONS] CONTAINER [CONTAINER...]`  
- 常用参数,详见 **[runoob][update]**

[update]:https://www.runoob.com/docker/docker-update-command.html
#### **docker cp**
- 含义:拷贝文件 从[SRC_PATH]到[DEST_PATH]  
- 语法:  
`docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH`  
`docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH`
# Docker镜像仓库相关操作
#### **docker commit**
 - 含义:将容器的当前状态保存为一个新的 Docker 镜像。
 - 语法:  
`docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`  
- 常用参数:
```
-a :提交的镜像作者。
-m :提交时的说明文字。
-p :提交镜像前暂停容器（默认为 true）。
```
#### **docker search**
- 含义:在 Docker Hub 或其他注册表中搜索镜像。  
- 语法:
`docker search [OPTIONS] TERM`
#### **docker push**
- 含义:将本地构建的 Docker 镜像推送到 Docker 注册表
- 语法:
`docker push [OPTIONS] NAME[:TAG]`
#### **docker pull**
- 含义: 从 Docker 注册表中拉取镜像到本地。
- 语法:
`docker pull [OPTIONS] NAME[:TAG|@DIGEST]`
# Docker本地镜像管理
#### **docker images**
- 含义:列出本地的 Docker 镜像
- 语法:  
`docker images [OPTIONS] [REPOSITORY[:TAG]]`
- 常用参数:
```
-a, --all: 显示所有镜像（包括中间层镜像）。
--digests: 显示镜像的摘要信息。
-f, --filter: 过滤输出，基于提供的条件。
--format: 使用 Go 模板格式化输出。
--no-trunc: 显示完整的镜像 ID。
-q, --quiet: 只显示镜像 ID。
```
#### **docker save/load**
- 含义:  
 ` docker save `命令用于将一个或多个 Docker 镜像保存到一个 tar 归档文件中。`docker load`命令用于从由 docker save 命令生成的 tar 文件中加载 Docker 镜像。
 - 语法:  
`docker save [OPTIONS] IMAGE [IMAGE...]`  
`docker load [OPTIONS]`
- 常用参数:  
  
`对于docker load`
```
-i, --input: 指定输入文件的路径。
```
`对于docker save`
```
-o, --output: 指定输出文件的路径。
```
- 实例  
  将 myimage:latest 镜像保存为 myimage.tar 文件。  
  `docker save -o myimage.tar myimage:latest`  
  加载镜像:  
`  docker load -i myimage.tar  `  
#### **docker history**
- 含义:查看指定镜像的历史层信息
- 语法:   
`docker history [OPTIONS] IMAGE`  
- 常用参数:
```
--no-trunc: 显示完整的输出，不截断信息。
-q, --quiet: 仅显示镜像 ID。
```
#### **docker build**
- 含义:docker build 命令通过读取 Dockerfile 中定义的指令，逐步构建镜像，并将最终结果保存到本地镜像库中。
- 语法:  
`docker build [OPTIONS] PATH | URL | -`
- 参数说明:
```
PATH: 包含 Dockerfile 的目录路径或 .（当前目录）。
URL: 指向包含 Dockerfile 的远程存储库地址（如 Git 仓库）。
-: 从标准输入读取 Dockerfile。
```
- 常用参数:
```
-t, --tag: 为构建的镜像指定名称和标签。
-f, --file: 指定 Dockerfile 的路径（默认是 PATH 下的 Dockerfile）。
--force-rm: 无论构建成功与否，一律删除中间容器。
--pull: 始终尝试从注册表拉取最新的基础镜像。
```
#### **docker tag**
- 含义:用于创建本地镜像的别名（tag），通过为镜像打标签
- 语法:  
`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`
#### **docker rmi**
- 含义:用于删除一个或多个 Docker 镜像。
- 语法:  
`docker rmi [OPTIONS] IMAGE [IMAGE...]`
- 常用参数:
```
-a, --all-tags: 指定仓库名称时，删除该仓库下的所有镜像。
-f, --force: 强制删除镜像，即使该镜像被容器使用。
--no-prune: 不删除悬空的父镜像。
-q, --quiet: 安静模式，不显示删除镜像的详细信息。
```
- 拓展:
`
docker rmi -d
`
会删除悬空镜像