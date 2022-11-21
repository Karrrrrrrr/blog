---
title: ZhParser
date: 2022-05-20T00:53:09+08:00

cover: https://pic4.zhimg.com/80/v2-97af21b844d39fdf35a0d5115e2f7cd7_720w.jpg
categories: ["代码", "数据库","插件"]
tags: [ "数据库","插件", "中文分词" ,"搜索引擎" ]
---

# 介绍
zhparser是Postgres的一个插件
可以用来分词,从而进行搜索
详情参照[文档](http://www.postgres.cn/docs/14/textsearch-controls.html#TEXTSEARCH-PARSING-DOCUMENTS) 

```yaml
# docker-compose.yml

version: "3.8"
services:
  postgres14_zhparser:
    image: 438577872/postgres_zhparser:14 # 14-alpine3.15或者用这个最小编译版 压缩后只有90M
    ports:
      - "5432:5432"
    container_name: postgres_zhparser
    environment:
      POSTGRES_PASSWORD: root
    volumes:
#      D://这里换成本地的目录
      - D://:/var/lib/postgresql/data
```

初始化数据库 执行以下sql
```sql
CREATE EXTENSION zhparser;
CREATE TEXT SEARCH CONFIGURATION chinese_zh (PARSER = zhparser);
ALTER TEXT SEARCH CONFIGURATION chinese_zh ADD MAPPING FOR n,v,a,i,e,l WITH simple;
```


测试插件是否安装成功SQL：
```postgresql
select ts_debug('chinese_zh', '白垩纪是地球上海陆分布和生物界急剧变化、火山活动频繁的时代');
```
```postgresql
CREATE TEXT SEARCH CONFIGURATION chinese_parser (PARSER = zhparser);
ALTER TEXT SEARCH CONFIGURATION chinese_parser ADD MAPPING FOR n,v,a,i,e,l,j WITH simple;

    
create table books
(
    book_name varchar default '',
    author    varchar default '',
    id        serial primary key
);
ALTER TABLE books  ADD COLUMN book_name_index tsvector; -- 添加一个分词字段


CREATE TRIGGER trigger_name
    BEFORE INSERT OR UPDATE
    ON videos
    FOR EACH ROW
EXECUTE PROCEDURE
    tsvector_update_trigger(book_name_index , 'public.chinese_parser', book_name );



insert into books values
    ('我在第一次见面的地方向她求婚了'),
    ('用代码分析了自己的6万条弹幕和评论，蚌埠住了'),
    ('优秀的程序员必须要知道的操作系统、网络IO、TCPIP协议、Socket通信、HTTPS加密算法、计算机组成原理双清华大佬36个小时给你全部讲清'),
    ('《孤勇者》前方核能！谁说女生不适合唱这歌');
UPDATE books SET book_name_index = to_tsvector('chinese_zh', coalesce(book_name, ''));--   // 将字段的分词向量更新到新字段中

CREATE INDEX idx_gin ON books USING GIN (book_name_index);
select * from books where book_name_index @@ to_tsquery('chinese_zh','程序员操作系统');
select * from books where book_name_index @@ plainto_tsquery('chinese_zh','程序员操作系统');

-- 排序 高亮结果 但是结果返回会比较慢  所以最好limit个数
select id,
       title,
       ts_rank_cd(information.abstract_index, query) as rank,
       ts_headline('chinese_zh',
                   information.abstract,
                   query, 'HighlightAll = true') as abstract
from information,
     plainto_tsquery('chinese_zh', '师范') as query
where
	information.abstract_index @@ query 
order by rank desc;

--  tsquery里面不能有空格  
```

用来修改分词的力度
```nginx
# postgresql.conf
zhparser.multi_short = true
zhparser.multi_duality = true
zhparser.multi_zmain = true
zhparser.multi_zall = false

```

编译mini版本必须用同版本的pg才行

## mini版本编译
```text
FROM postgres:15 as builder

RUN echo "deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free\n# deb-src http://mirrors.ustc.edu.cn/debian stable main contrib non-free\ndeb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free\n# deb-src http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free\n# deb http://mirrors.ustc.edu.cn/debian stable-proposed-updates main contrib non-free\n# deb-src http://mirrors.ustc.edu.cn/debian stable-proposed-updates main contrib non-free" > /etc/apt/sources.list && \
    apt-get update && apt-get install -y --no-install-recommends bzip2 gcc make libc-dev postgresql-server-dev-$PG_MAJOR wget  unzip  ca-certificates openssl && rm -rf /var/lib/apt/lists/*   && wget -q -O - "http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2" | tar xjf -   && wget -O zhparser.zip "https://ghproxy.com/https://github.com/amutu/zhparser/archive/master.zip"   && unzip zhparser.zip   && cd scws-1.2.3   && ./configure   && make install   && cd /zhparser-master   && SCWS_HOME=/usr/local make && make install

FROM postgres:15-alpine3.16
COPY --from=builder /usr/lib/postgresql/15/lib/bitcode/zhparser                             /usr/local/lib/postgresql/bitcode/zhparser/
COPY --from=builder /usr/lib/postgresql/15/lib/bitcode/zhparser/zhparser.bc                 /usr/local/lib/postgresql/bitcode/zhparser/zhparser.bc
COPY --from=builder /usr/lib/postgresql/15/lib/zhparser.so                                  /usr/local/lib/postgresql/zhparser.so
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--unpackaged--1.0.sql        /usr/local/share/postgresql/extension/zhparser--unpackaged--1.0.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.1--2.2.sql               /usr/local/share/postgresql/extension/zhparser--2.1--2.2.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--1.0--2.0.sql               /usr/local/share/postgresql/extension/zhparser--1.0--2.0.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.1.sql                    /usr/local/share/postgresql/extension/zhparser--2.1.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.0--2.1.sql               /usr/local/share/postgresql/extension/zhparser--2.0--2.1.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--1.0.sql                    /usr/local/share/postgresql/extension/zhparser--1.0.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser.control                     /usr/local/share/postgresql/extension/zhparser.control
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.0.sql                    /usr/local/share/postgresql/extension/zhparser--2.0.sql
COPY --from=builder /usr/local/lib/libscws.la                                               /lib/libscws.la
COPY --from=builder /usr/local/lib/libscws.so.1.1.0                                         /lib/libscws.so.1.1.0
COPY --from=builder /usr/share/postgresql/15/tsearch_data/dict.utf8.xdb                     /usr/local/share/postgresql/tsearch_data/dict.utf8.xdb
COPY --from=builder /usr/share/postgresql/15/tsearch_data/rules.utf8.ini                    /usr/local/share/postgresql/tsearch_data/rules.utf8.ini
RUN ln -s /lib/libscws.so.1.1.0 /lib/libscws.so.1 && ln -s /lib/libscws.so.1.1.0 /lib/libscws.so && echo "CREATE EXTENSION pg_trgm;CREATE EXTENSION zhparser;CREATE TEXT SEARCH CONFIGURATION chinese_zh (PARSER = zhparser);ALTER TEXT SEARCH CONFIGURATION chinese_zh ADD MAPPING FOR n,v,a,i,e,l,t WITH simple;" > /docker-entrypoint-initdb.d/init-zhparser.sql
CMD ["postgres"]
```

## deb版本编译
```text
FROM postgres:15 as builder

RUN echo "deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free\n# deb-src http://mirrors.ustc.edu.cn/debian stable main contrib non-free\ndeb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free\n# deb-src http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free\n# deb http://mirrors.ustc.edu.cn/debian stable-proposed-updates main contrib non-free\n# deb-src http://mirrors.ustc.edu.cn/debian stable-proposed-updates main contrib non-free" > /etc/apt/sources.list && \
    apt-get update && apt-get install -y --no-install-recommends bzip2 gcc make libc-dev postgresql-server-dev-$PG_MAJOR wget  unzip  ca-certificates openssl && rm -rf /var/lib/apt/lists/*   && wget -q -O - "http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2" | tar xjf -   && wget -O zhparser.zip "https://ghproxy.com/https://github.com/amutu/zhparser/archive/master.zip"   && unzip zhparser.zip   && cd scws-1.2.3   && ./configure   && make install   && cd /zhparser-master   && SCWS_HOME=/usr/local make && make install

FROM postgres:15
COPY --from=builder /usr/lib/postgresql/15/lib/bitcode/zhparser                             /usr/lib/postgresql/15/lib/bitcode/zhparser
COPY --from=builder /usr/lib/postgresql/15/lib/bitcode/zhparser/zhparser.bc                 /usr/lib/postgresql/15/lib/bitcode/zhparser/zhparser.bc
COPY --from=builder /usr/lib/postgresql/15/lib/bitcode/zhparser.index.bc                    /usr/lib/postgresql/15/lib/bitcode/zhparser.index.bc
COPY --from=builder /usr/lib/postgresql/15/lib/zhparser.so                                  /usr/lib/postgresql/15/lib/zhparser.so
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--unpackaged--1.0.sql        /usr/share/postgresql/15/extension/zhparser--unpackaged--1.0.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.1--2.2.sql               /usr/share/postgresql/15/extension/zhparser--2.1--2.2.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--1.0--2.0.sql               /usr/share/postgresql/15/extension/zhparser--1.0--2.0.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.1.sql                    /usr/share/postgresql/15/extension/zhparser--2.1.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.0--2.1.sql               /usr/share/postgresql/15/extension/zhparser--2.0--2.1.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--1.0.sql                    /usr/share/postgresql/15/extension/zhparser--1.0.sql
COPY --from=builder /usr/share/postgresql/15/extension/zhparser.control                     /usr/share/postgresql/15/extension/zhparser.control
COPY --from=builder /usr/share/postgresql/15/extension/zhparser--2.0.sql                    /usr/share/postgresql/15/extension/zhparser--2.0.sql
COPY --from=builder /usr/local/lib/libscws.la                                               /usr/local/lib/libscws.la
COPY --from=builder /usr/local/lib/libscws.so                                               /usr/local/lib/libscws.so
COPY --from=builder /usr/local/lib/libscws.so.1.1.0                                         /usr/local/lib/libscws.so.1.1.0
COPY --from=builder /usr/local/lib/libscws.so.1                                             /usr/local/lib/libscws.so.1
COPY --from=builder /usr/share/postgresql/15/tsearch_data/dict.utf8.xdb                     /usr/share/postgresql/15/tsearch_data/dict.utf8.xdb
COPY --from=builder /usr/share/postgresql/15/tsearch_data/rules.utf8.ini                    /usr/share/postgresql/15/tsearch_data/rules.utf8.ini

CMD ["postgres"]
```