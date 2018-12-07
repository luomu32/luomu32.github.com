# Dockerfile最佳实践

Dockerfile用于构建Docker镜像。Dockerfile是包含了所有构建指定镜像所需的有序的指令的文本文件。在Dockerfile中支持的指令参见：[Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)。

Docker镜像包含多个只读层，每个指令代表一层，一层一层堆叠成一个镜像。

```dockerfile
FROM ubuntu:15.04
COPY ./app
RUN make /app
CMD python /app/app.py
```

每个指令构建一层：

- FROM从ubuntu镜像创建一层
- COPP从Docker客户端所在当前目录拷贝文件
- RUN使用make构建你的应用程序
- CMD指定命令用于启动容器

当你启动一个容器时，在镜像之上还会创建一个可读写的层，称之为”容器层“。所有在容器运行时的变化，如写入一个新文件，修改已存在的文件，删除文件，都会写到这个可读写到容器层中。

## 一般准则和建议

###创建轻量级（ephemeral）容器

### 了解构建上下文

###管道`stdin`

