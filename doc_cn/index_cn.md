leveldb
=======

_Jeff Dean, Sanjay Ghemawat_

**leveldb**提供了键值对的持久存储。键和值都可以是任意的字节数组。键根据用户指定的比较函数进行排序。

## 打开一个数据库
一个**levledb**数据库对应一个文件系统目录。数据库的所有内容都保存在这个目录中。下面的示例展示了如何创建（如果不存在）和打开一个数据库：

```cpp
#include <cassert>
#include "leveldb/db.h"

leveldb::DB* db;
leveldb::Options options;
options.create_if_missing = true; // 数据库不存在则创建
leveldb::Status status = leveldb::DB::Open(options, "/tmp/testdb", &db); //"/tmp/testdb"为数据库存放目录
assert(status.ok());
...
```

如果您希望程序在发现数据库存在时报错，那么请在执行`leveldb::DB::Open`方法前添加下面的设置：

```cpp
options.error_if_exists = true;
```

## 状态 (Status)

您可能已经注意到上面的`leveldb::Status`类型了。**leveldb**中大多数可能遇到错误的函数都会返回这种类型的值。您可以检查这样的结果是否正确，并打印相关的错误消息：

```cpp
leveldb::Status s = ...;
if (!s.ok()) cerr << s.ToString() << endl;
```

