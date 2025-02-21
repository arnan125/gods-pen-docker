# godspen On-Premise

## WHAT

本项目提供了 码良 的私有部署（On-Premise）。基于 docker 及 docker-compose，部署你自己的 码良 服务。

码良 依赖 es、redis、mysql、邮件、oss服务，其中 es、redis、mysql 默认由docker容器提供，见 docker-compose.yaml。

## HOW

### 必要条件

- linux 系统或其他 linux 发行版，或者 macOS
- docker
- docker-compose
- 至少 3GB 内存，10G 可用存储空间 （如不使用 docker 容器提供 es、redis、mysql 全部或部分服务，可适当减少。单个es节点占用约1GB内存，默认启动了两个）

### 配置

项目提供了两个配置文件 config.yaml 和 nginx.conf。

config.yaml 集中了 码良 所有依赖服务的配置，如 redis、mysql 等。其中 redis、mysql、es 服务由相应 docker 容器提供，保持默认配置即可（也可自行配置相关字段，并在 docker-compose.yaml 移除对应服务）；邮件、对象存储不由 docker 容器提供，必须自行填写对应配置（可自建或使用第三方服务）。

nginx.conf 整合了 码良 内部的多个服务，是最终交付服务的实际入口。默认包含了最简配置，静态文件服务（含 html5 history 模式的支持，缓存）和api服务的反向代理，如需进行域名绑定等操作，可自行修改该文件进行配置。

### 构建本地镜像

完成配置以后，需要构建本地镜像。在项目根目录下运行 `make build` 或 `docker-compose build` 即可，构建过程耗时约4分钟，请耐心等待。

首次构建后，除非 config.yaml 再次改变，否则无需再次构建。


### 启动服务

完成本地镜像构建以后就可以启动服务了。在项目根目录下运行 `make start-server` 或 `docker-compose up -d` ，服务被依次启动。

启动服务的过程耗时约3分钟，可用 `docker logs <container id>` 查看运行情况。

启动成功后，访问地址为`http://<host:port>/admin`，如使用了 nginx.conf 默认配置，访问地址即 http://127.0.0.1/admin

### 中止服务

`make stop-server` 。暂停服务，不会删除现有容器，可通过 `make start-server` 再次恢复运行。

### 终止服务

`make remove-server` 。移除服务，会删除现有容器，可通过 `make start-server` 重新创建容器启动服务。

## troubleshooting

### es服务停止

运行 `docker ps -a` 检查容器运行状态，发现所有 es 容器处于退出状态（exited），挨个查看 es 容器的日志, `docker logs es`，发现如下log

<pre>
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2019-04-30T02:43:52,717][INFO ][o.e.n.Node               ] [gKecOlD] stopping ...
[2019-04-30T02:43:52,971][INFO ][o.e.n.Node               ] [gKecOlD] stopped
[2019-04-30T02:43:52,971][INFO ][o.e.n.Node               ] [gKecOlD] closing ...
[2019-04-30T02:43:53,041][INFO ][o.e.n.Node               ] [gKecOlD] closed
</pre>

据es文档

> Elasticsearch 对各种文件混合使用了 NioFs（ 注：非阻塞文件系统）和 MMapFs （ 注：内存映射文件系统）。请确保你配置的最大映射数量，以便有足够的虚拟内存可用于 mmapped 文件。这可以暂时设置：   
`sysctl -w vm.max_map_count=262144`   
或者你可以在 `/etc/sysctl.conf` 通过修改 `vm.max_map_count` 永久设置它   