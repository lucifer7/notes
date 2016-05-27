# Redis in action

## 1. Download redis.py

[redis-py](https://github.com/andymccurdy/redis-py#connection-pools)
[redis实战](http://redisinaction.com/index.html)


1. git clone https://github.com/andymccurdy/redis-py.git
2. cd redis-py
3. python setup.py install


## 2. log counter

``` python

PRECISION = [1, 5, 60, 300, 3600, 18000, 86400]

def update_counter(name, count=1, now=None, conn=None):
    conn = conn or redis.StrictRedis(host='localhost', port=6379, db=0)
    now = now or time.time()
    pipe = conn.pipeline()
    for prec in PRECISION:
        pnow = int(now / prec) * prec
        hash = "%s:%s" % (prec, name)
        pipe.zadd("known:", 0, hash)
        pipe.hincrby("count:" + hash, pnow, count)
    pipe.execute()

```

- use hash to save counts
- user zset to save all keys


关键的是如何使用后台线程去清理






- python map()

```python
 _ret = map(lambda a, b: str(a) + "->" + str(b), [1, 2, 3], ['a', 'b', 'c'])
```


# 3. 使用redis实现分布锁

1. 简易分布锁

>mysql 中的事务1

    






