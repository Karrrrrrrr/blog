---
title: "ElasticSearch"
date: 2024-08-10T14:00:00+08:00


categories: ["代码", "搜索引擎", "elasticsearch" ]

tags: ["elasticsearch", "搜索引擎" ]
---
```yaml
#config/elasticsearch.yaml
# 配置host为0.0.0.0 允许远程访问
http.host: 0.0.0.0
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
# 开启密码访问
xpack.security.enabled: false
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  es:
    container_name: es
    image: elasticsearch:7.17.5
    environment:
      - discovery.type=single-node
      - TZ=Asia/Shanghai
    ports:
      - "9200:9200"
      - "9300:9300"

    volumes:
      - ./config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./data:/usr/share/elasticsearch/data
      - ./plugins:/usr/share/elasticsearch/plugins
    networks:
      - proxy
networks:
  proxy:
    external: true
```
## 手动安装依赖
```shell
version="8.4.1"
wget https://get.infini.cloud/elasticsearch/analysis-ik/${version} -O ./plugins/analysis-ik.zip
unzip analysis-ik.zip -d plugins/ik
```

# 总结
下面的配置可以看情况改
```shell
rm -rf config 
rm -rf data

mkdir -p config
mkdir -p data 

version="8.12.2"
cat > config/elasticsearch.yml << EOF
# 配置host为0.0.0.0 允许远程访问
http.host: 0.0.0.0
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
# 开启密码访问
xpack.security.enabled: false
EOF


cat > docker-compose.yml << EOF
# docker-compose.yml
version: '3.8'
services:
  es:
    container_name: es
    image: swr.cn-north-4.myhuaweicloud.com/kar/elasticsearch:${version}
    environment:
      - discovery.type=single-node
      - TZ=Asia/Shanghai
    ports:
      - "9200:9200"
      - "9300:9300"

    volumes:
      - ./config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./data:/usr/share/elasticsearch/data
      - ./plugins:/usr/share/elasticsearch/plugins
#    networks:
#      - proxy
#networks:
#  proxy:
#    external: true
EOF

#wget https://get.infini.cloud/elasticsearch/analysis-ik/${version} -O analysis-ik-${version}.zip
wget "https://alist.single-one.top/d/opt/alist/data/alicloud_open/Downloads/ik/analysis-ik-${version}.zip?sign=rsFW-ZVGxsdNw5q6sBwYKcFtEvDIOj2hoexkaD0FGqI=:0" -O analysis-ik-${version}.zip
rm -rf plugins/ik
mkdir -p plugins/ik
unzip analysis-ik-${version}.zip -d plugins/ik
rm analysis-ik-${version}.zip
docker-compose up -d
```