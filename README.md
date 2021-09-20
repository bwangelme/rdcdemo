go-redis Client 源码阅读
====

## 连接池

+ `options.PoolSize` 可以指定连接池中最大的连接数，默认是 `runtime.GOMAXPROCS * 10`
+ `options.MinIdleConns` 指定连接池中最少的空闲连接数

## 连接的建立和删除

1. 每次执行 Redis 命令的时候，会创建一个 [Cmd](command.go:184) 对象，然后将这个对象传递给 Redis 客户端中的 `cmdable` 属性函数，由它来执行具体的操作
2. 获取和释放连接在 [withConn](redis.go:271) 函数中进行，连接池中的连接对象并未进行过初始化，执行了 [initConn](redis.go:214) 函数的连接后，才真正和 redis 服务器建立了连接。
3. cmdable 函数执行完成后，会释放连接会连接池中。具体逻辑在 [releaseConn](redis.go:259) 函数中