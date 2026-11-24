# 校园外卖业务的 Redis 数据类型选型

| 业务场景 | 最适合的数据类型 | 理由 | 示例命令 |
| --- | --- | --- | --- |
| 缓存菜品详情 | String | 将菜品详情序列化为 JSON，整条读取，并可为缓存键设置过期时间。 | `SET lab:takeout:dish:d101:detail '{"id":"d101","name":"宫保鸡丁饭","price":18}' EX 300` |
| 购物车里的商品清单 | List | 本题按加入顺序保存商品条目，用尾部追加和范围查询展示清单。 | `RPUSH lab:takeout:cart:u1001 d101 d102`；`LRANGE lab:takeout:cart:u1001 0 -1` |
| 用户收藏去重 | Set | 菜品 ID 唯一，重复收藏不会增加成员，可查询收藏数量或是否已收藏。 | `SADD lab:takeout:favorites:u1001 d101 d101 d103`；`SCARD lab:takeout:favorites:u1001` |
| 热搜菜品榜 | ZSet 有序集合 | 菜品 ID 为成员、热度为分数，支持热度增量更新和按分数倒序取榜。 | `ZADD lab:takeout:search:hot 30 d101 45 d102 20 d103`；`ZRANGE lab:takeout:search:hot 0 2 REV WITHSCORES` |
| 存一条订单的多个字段 | Hash | 一个订单键对应多个字段，能单独读取或更新状态、用户和金额等属性。 | `HSET lab:takeout:order:o1001 userId u1001 status CREATED totalAmount 36`；`HGETALL lab:takeout:order:o1001` |

购物车这一项只描述有顺序的商品条目清单；如果业务还要求按菜品 ID 合并、修改购买数量，则应采用 Hash 保存“菜品 ID → 数量”。

菜品详情使用普通 String 保存 JSON 文本，不需要 RedisJSON 扩展。销量计数使用 `INCR`，与热搜榜的热度统计分开保存。销量榜的初始化分数和三个计数器保持一致，后续用 `MULTI`、`INCR`、`ZINCRBY`、`EXEC` 一起更新。
