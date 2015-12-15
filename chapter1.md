# CQL 数据模型

## 数据模型概念
## 数据模型示例
## 音乐服务示例

一个社交音乐服务要求 songs 表有 title, album(相册) 和一个 artist 列，外加一个存储音频文件的列（叫做 data）。songs 表使用 uuid 作为主键。

```
CREATE TABLE songs (
  id uuid PRIMARY KEY,
  title text,
  album text,
  artist text,
  data blob
);
```

在关系型数据库中，你也许会创建一个 playlists 表，它包括一个指向 song 表的外键，但是在 cassandra，join 操作并不适用于分布式系统。为了描绘 playlists 的数据，你可以这样创建表：

```
CREATE TABLE playlists (
  id uuid,
  song_order int,
  song_id uuid,
  title text,
  album text,
  artist text,
  PRIMARY KEY  (id, song_order )
);
  
```
在 playlists 表中，id 和 song_order 的组合可以唯一的标示表中的一行数据。你可以有多行具有相同 id 的数据，只要这些行包含不同的 song_order。

以下是插入单条记录到 playlists 表的例子：

```
INSERT INTO playlists (id, song_order, song_id, title, artist, album)
  VALUES (62c36092-82a1-3a00-93d1-46196ee77204, 1,
  a3e64f8f-bd44-4f28-b8d9-6938726e34d4, 'La Grange', 'ZZ Top', 'Tres
Hombres');
INSERT INTO playlists (id, song_order, song_id, title, artist, album)
  VALUES (62c36092-82a1-3a00-93d1-46196ee77204, 2,
  8a172618-b121-4136-bb10-f665cfc469eb, 'Moving in Stereo', 'Fu Manchu',
 'We Must Obey');
INSERT INTO playlists (id, song_order, song_id, title, artist, album)
  VALUES (62c36092-82a1-3a00-93d1-46196ee77204, 3,2b09185b-fb5a-4734-9b56-49077de9edbf, 'Outside Woman Blues', 'Back Door
Slam', 'Roll Away');
```

插入数据之后，可以使用 SELECT 查询语句展示数据：

```SELECT album, title FROM playlists WHERE artist = 'Fu Manchu';```

![](http://docs.datastax.com/en/cql/3.1/cql/images/select_playlists.png)
 
下面的例子介绍了如何使用 artist 作为过滤器筛选数据：

```SELECT album, title FROM playlists WHERE artist = 'Fu Manchu';```

Cassandra 会拒绝这个查询，因为查询要求顺序扫描整个表，但 artist 并不是一个分区键（partition key）或聚集列（clustering column)。通过在 artist 列上创建一个索引，Cassandra 可以获取数据。

```CREATE INDEX ON playlists( artist );```

现在，你可以通过 Fu Manchu 查询到响应的歌曲：

![](http://docs.datastax.com/en/cql/3.1/cql/images/artist_fumanchu.png)