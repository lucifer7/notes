# Python

## 帮助文档
[谷歌风格目录](http://zh-google-styleguide.readthedocs.org/en/latest/google-objc-styleguide/contents/)
[帮助手册]()


[TOC]





## 操作mysql

安装: `sudo apt-get install python-mysqldb` 

1. **get**方法:

    ```python
    def get(cursor, sql):
    try:
        # get column names
        cursor.execute(sql)
        column_names = [d[0] for d in cursor.description]
        return [dict(itertools.izip(column_names, row)) for row in cursor][0]
    except MySQLdb.OperationalError:
        raise
    ```

2. **get many records**方法:
	```python
def get_many(cursor, sql):
    """
        sql 查询数据 返回 dict格式
    :param cursor:
    :param sql:
    :return:
    """
    try:
        # get column names
        cursor.execute(sql)
        column_names = [d[0] for d in cursor.description]
        return [Row(itertools.izip(column_names, row)) for row in cursor]
    except MySQLdb.OperationalError:
        raise
    ```



##操作 itertools
- 首先认识python 迭代器

```python
if __name__ == '__main__':
    # check_bidding_config()
    lst = range(2)
    it = iter(lst)
    try:
        while True:
            print (str(it.next()))
    except:
        pass

    #已经迭代完了
    it = iter(lst)

    for i in it:
        print i
```
> 1. 发现python 使用异常来终止迭代
> 2. 所有的迭代器都可以用 `for i in it`


- 工具类itertools
返回迭代器的各种方法，非常强大
    1. **izip**

    ```python
    def get(cursor, sql):
        try:
            # get column names
            cursor.execute(sql)
            column_names = [d[0] for d in cursor.description]
            #izip 返回的是个组合的迭代器
            return [dict(itertools.izip(column_names, row)) for row in cursor][0]
        except MySQLdb.OperationalError:
            raise

	```

