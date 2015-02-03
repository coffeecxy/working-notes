zoom 设计实现
===

在[zoom的介绍](zoom.md)中给出了zoom.us提供的api的方式。

设计的方案要实现的功能有下面的

## 验证请求
对过来的请求进行判断，保证是从一个服务器过来的，而且其必须给出需要的`api_key`和`api_secret`，这个两个字段就使用zoom提供给我们的

## 创建一个Meeting

/meeting/getOne

