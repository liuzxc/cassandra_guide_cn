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

在关系型数据库中，你也许会创建一个 playlists 表，它包括一个指向 song 表的外键，但是在 cassandra，join 操作并不适用于分布式系统。
