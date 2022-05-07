---
title: zhparser
cover: https://pic4.zhimg.com/80/v2-97af21b844d39fdf35a0d5115e2f7cd7_720w.jpg
---

zhparser是Postgres的一个插件
可以用来分词,从而进行搜索


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
```sql
select ts_debug('chinese_zh', '白垩纪是地球上海陆分布和生物界急剧变化、火山活动频繁的时代');
```
```sql
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
