# NoSQL

- 按模式批量删除redis数据项：`bin/redis-cli -a 'password' --raw KEYS "cron_task_status__*" | xargs -d '\n' bin/redis-cli -a 'password' DEL`

## 值得关注

- [Beansdb](https://github.com/douban/beansdb)

