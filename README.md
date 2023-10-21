## 执行dnsmonster，将dns数据插入到clickhouse

```shell
./cmddnsmonster --devName eth0 --packethandlercount=8 --useafpacket --clickhouseaddress=127.0.0.1:8080  --clickhousedatabase=default --clickhouseoutputtype=1
```

此处clickhouseaddress地址是由nginx代理的clickhouse地址，clickhouse实际地址是127.0.0.1:9000

## 部署到服务器时，两个注意事项

1. clickhouse的dns_dictionary.xml需放到clickhouse serve的配置目录下，目录路径在config.xml的

```xml
<dictionaries_config>*_dictionary.xml</dictionaries_config>
```

配置块中查看。
2. dictionaries下的csv文件需部署到相应录下，目录位置可在dns_dictionary.xml的

```xml
<path>/opt/dictionaries/dns_response.tsv</path>
```

配置块中查看。
3. 提前将tables.sql导入到clickhouse。

## nginx配置

```nginx
user  nginx;
worker_processes  auto;
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

stream {
    server {
        listen 8080;
        proxy_connect_timeout 5s;
        proxy_timeout 20s;
        proxy_pass clickhouse:9000;
    }
    log_format  main  '$remote_addr - [$time_local] '
                      '$status';

    access_log  /var/log/nginx/access.log  main;
}
```
