---
type: permanent
---
# 🌐 核心观点

用一两句话概括这个原子化的想法.

---

# 🔖 详细解释

## Redis

```bash
docker run -it \
-v ~/Documents/volumes/redis_conf/redis_master/conf:/usr/local/etc/redis \
--name redis-master \
-p 6379:6379 \
-d redis \
redis-server /usr/local/etc/redis/redis.conf
```

## MySQL

```bash
docker run \
-p 3306:3306 \
--restart=always \
--name mysql \
--privileged=true \
-e MYSQL_ROOT_PASSWORD=my-secret-pw \
-d mysql:8

-v /home/mysql/log:/var/log/mysql \
-v /home/mysql/data:/var/lib/mysql \
-v /home/mysql/conf/my.cnf:/etc/mysql/my.cnf \
```

## ES 相关部署

#### 配置启动 ElasticSearch

拉取镜像

```bash
docker pull elasticsearch:7.17.9
```

在配置 volume 之前需要运用 docker cp 命令将数据复制到文件夹中，否则会启动失败：

```bash
docker cp 29507f11f773:/usr/share/elasticsearch/data /Users/bytedance/Documents/volumes/es
docker cp 29507f11f773:/usr/share/elasticsearch/config /Users/bytedance/Documents/volumes/es
```

使用 docker 启动 elasticsearch

```bash
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-e "ES_JAVA_OPTS=-Xms512m -Xmx1024m" \
-e "ingest.geoip.downloader.enabled=false" \
-v /Users/yuankangqing/Documents/volumes/es/plugins/:/usr/share/elasticsearch/plugins \
-v /Users/yuankangqing/Documents/volumes/es/data/:/usr/share/elasticsearch/data \
-v /Users/yuankangqing/Documents/volumes/es/logs/:/usr/share/elasticsearch/logs \
-v /Users/yuankangqing/Documents/volumes/es/config:/usr/share/elasticsearch/config \
-d elasticsearch:7.17.9
```

在 ES 配置文件中允许跨域请求：

```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 配置启动 Kibana

```bash
docker cp e7aab1c16542:/usr/share/kibana/config /Users/bytedance/Documents/volumes/kibana/config
```

```bash
docker run --name kibana \
--link=elasticsearch \
-p 5601:5601 \
-v /Users/yuankangqing/Documents/volumes/kibana/config/:/usr/share/kibana/config \
-d kibana:7.17.9
```

 在 Kibana 配置文件中设置中文：
 
```
 i18n.locale: "zh-CN"
```

## 配置启动 ElasticSearch-Head

```bash
docker run -d --name=elasticsearch-head -p 9100:9100 \
mobz/elasticsearch-head:5-alpine
```

进⼊ \_site ⽬录，修改 vendor.js ⽂件 6886 和 7574 ⾏ application/x-www-form-urlencoded 改成 application/json;charset=UTF-8
访问：[http://localhost:9100/](http://localhost:9100/)

---

# 📚 参考内容

