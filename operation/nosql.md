# NoSQL

- 按模式批量删除redis数据项：`bin/redis-cli -a 'password' --raw KEYS "cron_task_status__*" | xargs -d '\n' bin/redis-cli -a 'password' DEL`
- 但上面的批量删除命令，在遇到key中带特殊字符时可能无法成功删除，这时可以通过redis的EVAL命令以lua脚本的方式来删除：`EVAL "return redis.call('del', unpack(redis.call('keys', ARGV[1])))" 0 cron_task_status__*`，来源：http://stackoverflow.com/questions/4006324/how-to-atomically-delete-keys-matching-a-pattern-using-redis

## 值得关注

- [Beansdb](https://github.com/douban/beansdb)

