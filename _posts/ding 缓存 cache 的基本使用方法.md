# ding 缓存 cache 的基本使用方法

 1. ```cache.set(key, value, timeout=DEFAULT_TIMEOUT, version=None)```

    key 为字符串， value 为可pickable形式的python对象

 2. ```cache.get(key, default=None, version=None)```

    当get请求的key不存在时，则返回设置的这个值，而不是默认不存在返回的None

 3. ```cache.add(key, value, timeout=DEFAULT_TIMEOUT, version=None)```

    当add的key不存在时，则新建key，存在则不做任何操作，新建成功返回True，失败返回False

 4. ```cache.get_or_set(key, default, timeout=DEFAULT_TIMEOUT, version=None)```

    需要2个参数，第一个为key，第二个为key不存在时设置的值。意思是如果key存在，则返回key的值，如果不存在则给key设置一个值

 5. ```cache.get_many(keys, version=None)```

    通过传入一个keys列表，以字典格式返回这些列表中key存在的缓存值

 6. ```cache.set_many(dict, timeout)```

    通过传入字典，批量设置缓存

 7. ```cache.delete(key, version=None)```

    删除一个key

 8. ```cache.delete_many(keys, version=None)```

    批量删除key

 9. ```cache.clear()```

    清空缓存，需要注意的是这会删除缓存里的所有key，可能包含一些并不是你设置的key

 10. ```cache.touch(key, timeout=DEFAULT_TIMEOUT, version=None)```

     更新key的过期时间为timeout设置的值，timeout是可选的，如果不写则默认为设置的值

     更新成功则返回True，否则返回False

 11. ```cache.incr(key, delta=1, version=None)```

     incr递增一个已存在的int类型的key的值，默认情况下递增幅度为1，通过指定可以设置递增的幅度

 12. ```cache.decr(key, delta=1, version=None)```

     与incr递增类似，decr为递减

     