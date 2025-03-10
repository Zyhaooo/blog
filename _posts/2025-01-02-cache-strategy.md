---
title: 常见缓存策略
layout: post
---

### Cache-Aside

也叫旁路缓存,主要用于读写都比较频繁的场景

```sh
# 命中缓存
application <--- cache

# 未命中缓存
cache <--填充缓存-- application <--从db中获取数据-- db

# 写操作
application --> db
```

这种策略将缓存置于一边,应用程序需要显式的读写它,而不是靠另外的缓存系统来操作,这样做可以当缓存系统发生错误或者缓存穿透时,应用程序也能通过db来给应用程序提供服务

但是值得注意的是,写操作时,应用程序是直接写入数据库的,这就有可能导致缓存中的数据和数据库中的数据不一致.所以一般来说,这种策略下的缓存通过设置过期时间来保证缓存的一致性或者当写操作时删除相对应的缓存内容来达到缓存的一致性

### Read-Through Cache

也叫读穿透缓存,用于读操作远多于写操作的场景.特别是当希望简化应用程序逻辑并让缓存系统处理大部分数据读取和更新时.

```sh
# 命中缓存
application <--- cache

# 未命中缓存
application <--- [cache <--- db]

# 写操作
application --> [cache --> db]
```

读穿透和旁路缓存很类似,但是不同点在于,读穿透是通过独立的缓存系统来管理缓存,而旁路缓存是通过应用程序来管理缓存

### Write-Through Cache

也叫写穿透缓存,缓存写入时总是先更新缓存然后再去更新数据库.

```sh
#写操作
application --> cache --> db
```

写穿透保证了缓存的一致性,但是多了一步额外的写操作,会导致较高的写操作延迟.

通常读写穿透缓存配合使用

### Write-Around Cache

旁路写缓存,当应用程序写入数据的时候,直接写入数据库,而不主动更新缓存

这种缓存策略常用于只缓存部分热点数据的场景.

由于不会主动更新缓存,所以不可避免的出现缓存不一致的问题

### Write-Back Cache


### TODO

