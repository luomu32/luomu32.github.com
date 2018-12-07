# Docker

容器技术的出现，能解决环境不一致导致的问题。也方便部署，减少环境搭建的工作量。同时为DevOps提供助力，因为减少了运维的复杂度。也对持续交付有帮助。

## 镜像

分层。

镜像的下载和发布。私服。

镜像的构建：dockerfile

## 容器

以Java作比喻，镜像就像是类，定义了结构和功能，容器就是实例。根据镜像就能运行容器。

容器是无状态的，也就是说不应当把生成的数据保存在容器中，所以一般会指定数据卷，以便将数据保存到宿主机中。

`docker run -d -it --name xxx -p 3000:3000 -v /data/xxx -l xxx`。运行一个容器，一般需要指定容器名称，以便后续启停等维护；需要暴露的端口以便提供服务；需要挂载数据卷以便保存数据；需要指定镜像

`docker start`、`docker stop`、	`docker restart`。

`docker exec -it container id`

## 存储

## 网络

##日志

Docker容器默认将应用产生的日志输出到标准输出中。通过`docker logs container id`查看日志。缺点是没有持久化，也不分割，更没有搜索了。虽然Docker提供了LogDriver机制方便的替换实现，可以将日志保存到文件等之中，但这也不是一个很好的方案。由于容器的生命周期较为短暂，所以最好采用集中式日志平台，比如ELK。

Filebeat用于收集日志文件，然后转发给Logstash进行分析和过滤，也可以直接发送到Elasticsearch。

## 监控



##编排

Docker Compose

Swarm