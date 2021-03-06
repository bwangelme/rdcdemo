go-redis Client 源码阅读
====

> 基于 8.11.3 版本

## 连接池

+ `options.PoolSize` 可以指定连接池中最大的连接数，默认是 `runtime.GOMAXPROCS * 10`
+ `options.MinIdleConns` 指定连接池中最少的空闲连接数
+ `options.MaxRetries` 命令执行失败后的最大重试次数
+ `options.MinRetryBackoff` 重试之间的最小等待时间
+ `options.MaxRetryBackoff` 重试之间的最大等待时间
+ `options.DialTimeout` 连接超时
+ `options.ReadTimeout` 读超时
+ `options.WriteTimeout` 写超时
+ `options.MinIdleConns` 最小空闲连接数
+ `options.MaxConnAge` 连接的存活时间
+ `options.IdleTimeout` 空闲连接的存活时间

## 连接的建立和删除

vendor/github.com/go-redis/redis/v8/command.go
1. 每次执行 Redis 命令的时候，会创建一个 [Cmd](vendor/github.com/go-redis/redis/v8/command.go#L184) 对象，然后将这个对象传递给 Redis 客户端中的 `cmdable` 属性函数，由它来执行具体的操作
2. 获取和释放连接在 [withConn](vendor/github.com/go-redis/redis/v8/redis.go#L271) 函数中进行，连接池中的连接对象并未进行过初始化，执行了 [initConn](vendor/github.com/go-redis/redis/v8/redis.go#L283) 函数的连接后，才真正和 redis 服务器建立了连接。
3. cmdable 函数执行完成后，会释放连接会连接池中。具体逻辑在 [releaseConn](vendor/github.com/go-redis/redis/v8/redis.go#L271)

## 失败重试逻辑

1. `options.MaxRetries` 设定命令执行失败后的最大重试次数。
2. 重试之间等待的时间介于 `options.MinRetryBackoff` 和 `options.MaxRetryBackoff`
3. 重试之间的等待时间是指数增加的，具体逻辑在 [RetryBackoff](vendor/github.com/go-redis/redis/v8/internal/internal.go:9) 函数中，它用来获取重试之间的等待时间