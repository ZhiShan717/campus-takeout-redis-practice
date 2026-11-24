# 校园外卖 Redis 数据类型练习

`redis-practice.txt` 是可直接交给 `redis-cli` 执行的 Redis 命令文件，不含提示符、注释或输出文字。`data-type-selection.md` 包含五个业务场景的数据类型选型表和理由。

## 运行方式

在本机或服务器的 Linux Shell 中，进入本目录，连接专用的 Redis 7 实验实例执行：

```bash
redis-cli --no-raw < redis-practice.txt
```

如果 Redis 运行在名为 `redis-practice-lab` 的 Docker 容器中，则执行：

```bash
docker exec -i redis-practice-lab redis-cli --no-raw < redis-practice.txt
```

命令文件会重置自己使用的九个 `lab:takeout:` 实验键，不使用 `FLUSHDB` 或 `FLUSHALL`。建议在专用实验实例运行。菜品详情缓存的有效期为 300 秒。

## 题目一的实现

| 菜品 ID | 菜品名称 | 初始销量 | 增加后的销量 |
| --- | --- | ---: | ---: |
| d101 | 宫保鸡丁饭 | 12 | 13 |
| d102 | 番茄鸡蛋饭 | 8 | 8 |
| d103 | 红烧牛肉面 | 20 | 20 |

三个销量分别使用 String 保存，以 `INCR` 为 d101 增加 1。ZSet 的成员为菜品 ID，分数为对应销量。初始化使用相同销量值，后续把 `INCR` 与 `ZINCRBY` 放入同一个 `MULTI`/`EXEC` 事务。实验命令保证键类型正确且计数值为整数。

Redis 事务执行期间其他客户端命令不会插入，但不提供运行时错误回滚。因此不能将该示例理解为对任意错误都能自动回滚的事务。

`ZRANGE lab:takeout:sales:rank 0 2 REV WITHSCORES` 的 `0`、`2` 是包含端点的排名下标，结果按销量从高到低为：d103 20、d101 13、d102 8。分数相同的成员按逆字典顺序排列；本例分数互不相同。

## 题目二的实现

五个业务场景依次选择 String、List、Set、ZSet、Hash。购物车 List 的适用范围是按加入顺序展示商品条目；需要按菜品 ID 管理数量时选 Hash。完整说明见 `data-type-selection.md`。

## 提交材料

- `redis-practice.txt`：Redis 命令文件。
- `data-type-selection.md`：场景选型表。
- `任务原文.txt`：附件题目文字转写。
- 整体 Word 文档、三张原生终端截图和逐行文字保存在本地提交目录；命令已在 Redis 7.4.12 实际执行并完整回放验证。

## 官方参考

- Redis 数据类型：https://redis.io/docs/latest/develop/data-types/
- ZRANGE：https://redis.io/docs/latest/commands/zrange/
- Redis 事务：https://redis.io/docs/latest/develop/using-commands/transactions/
