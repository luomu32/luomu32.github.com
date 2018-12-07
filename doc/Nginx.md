# Nginx

Nginx是一款高性能的反向代理服务器。同类型的软件还有Apache Web Server、Squid等。

## 基础配置

常见的配置会包含以下几个配置项：

- http
- server
- upstream
- location

### upstream

### server

## WebSocket支持

使用HTTP/1.1的协议切换机制，来将客户端和服务器端之间的连接从HTTP/1.1切换到WebSocket。

从1.3.13开始，nginx实现了特殊操作，允许客户和服务器之间设定一个通道，当客户端的请求头中包含upgrade，同时服务器端返回101响应码

默认情况下，如果服务器在60s内未传送任何数据，连接就会被关闭。超时时间可以通过`proxy_read_timeout`来设定。另外，服务器可以定期发送一个WebSocket的ping帧来重制超时和检测连接是否已断开。

配置示例：

```nginx
location /chat/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

## HTTPS支持

