---
layout: post
title: "mysql-schema-sync的改进"
subtitle: '数据库表结构批量同步解决方案'
author: "陶叔"
header-style: text
tags:
- Go
- mysql-schema-sync
- Python
- 表结构同步
---

### 环境搭建Go&Python
1. brew install安装go环境
2. 安装调试python3环境
3. 下载代码：go get -u github.com/hidu/mysql-schema-sync

### Python脚本
```
#!/usr/bin/env python3
# coding=utf-8

import os
import json
import pymysql
import sys

def get_m_json(filepath, key, value):
    key_ = key.split(".")
    key_length = len(key_)
    with open(filepath, 'r') as f:
        json_data = json.load(f)
        i = 0
        a = json_data
        while i < key_length:
            if i + 1 == key_length:
                a[key_[i]] = "USERNAME:PASSWORD@(" + mysources + ":3306)/" + value
                print("mysources: ", mysources)
                i = i + 1
            else:
                a = a[key_[i]]
                i = i + 1
    f.close()
    return json_data


def get_s_json(filepath, key, value):
    key_ = key.split(".")
    key_length = len(key_)
    with open(filepath, 'r') as f:
        json_data = json.load(f)
        i = 0
        a = json_data
        while i < key_length:
            if i + 1 == key_length:
                a[key_[i]] = "usr:pwd@(" + mydest + ":3306)/" + value
                i = i + 1
            else:
                a = a[key_[i]]
                i = i + 1
    f.close()
    return json_data


def rewrite_json_file(filepath, json_data):
    with open(filepath, 'w') as f:
        json.dump(json_data, f)
    f.close()


if __name__ == '__main__':
    dest = sys.argv[1]
    mysources = "source-connection"
    mydest = "mysql-m-svc." + dest + ".svc.cluster.local"
    
    print('source:' + mysources)
    print('dest:' + mydest)
    json_path = "/root/go/src/github.com/hidu/mysql-schema-sync/config.json"
    mysqlsync = "/root/go/bin/mysql-schema-sync"
    print("jsonpath:", json_path)
    conn = pymysql.connect(host=mysources, port=3306, user="USERNAME", password="PASSWORD",
                           database="information_schema")
    cursor = conn.cursor()
    sql = "select SCHEMA_NAME from INFORMATION_SCHEMA.SCHEMATA where SCHEMA_NAME not in " \
          "('mysql','information_schema','performance_schema','test','sys');"
    cursor.execute(sql)
    m_dbs = cursor.fetchall()
    cursor.close()
    conn.close()
    print("sourcedbs:", m_dbs)

    conn = pymysql.connect(host=mydest, port=3306, user="usr", password="pwd", database="information_schema")
    cursor = conn.cursor()
    sql = "select SCHEMA_NAME from INFORMATION_SCHEMA.SCHEMATA where SCHEMA_NAME not in " \
          "('mysql','information_schema','performance_schema','test','sys');"
    cursor.execute(sql)
    s_dbs = cursor.fetchall()
    cursor.close()
    conn.close()
    print("destdbs:", s_dbs)

    for m_db in m_dbs:
        for s_db in s_dbs:
            if m_db == s_db:
                print("json_path:  ", json_path)
                print("m_db[0]: ", m_db[0])
                m_json_data = get_m_json(json_path, "source", m_db[0])

                rewrite_json_file(json_path, m_json_data)
                s_json_data = get_s_json(json_path, "dest", m_db[0])
                rewrite_json_file(json_path, s_json_data)

                print("s_json_data:  ", s_json_data)
                sync_cmd = str(mysqlsync) + ' -conf ' + json_path + ' -sync 2>/dev/null >> bat_to_' + dest + '_' + \
                           m_db[0] + '.sql'
                os.system(sync_cmd)
                sync_cmd1 = str(mysqlsync) + ' -conf ' + json_path + ' -sync 2>/dev/null >>sync' + dest + '.log'
                print(sync_cmd1)
                os.system(sync_cmd1)
                break
```

### 部署到服务器
> 192.168.XXX.XXX

### 查看SQL
> mysql-schema-sync -conf mydb_conf.json 2>/dev/null >db_alter.sql

### 脚本批量执行（某个环境的所有数据库）
> /usr/local/bin/python3 /root/go/all_db_sync.py sit6

### 定时任务crontab -e （每周日凌晨，执行所有环境的数据库）
```
0 1 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev1
10 1 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev2
20 1 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev3
30 1 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev4
40 1 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev5
50 1 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev6
0 2 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py dev7
10 2 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py sit5
20 2 * * 0 /usr/local/bin/python3 /root/go/all_db_sync.py sit6
```

### 存在问题
- 会删除测试环境新增的，但源数据源还没产生的表

### 简单解决：修改代码
- index.go
```
	if drop {
		// 注释掉drop table，不drop了
		//dropSQL := idx.alterDropSQL()
		//if dropSQL != "" {
		//	alterSQL = append(alterSQL, dropSQL)
		//}
	}
```

- schemaSync.go
``` 
if sSchema == "" {
    alter.Type = alterTypeDrop
    // 注释掉drop table，不drop了
    //alter.SQL = append(alter.SQL, fmt.Sprintf("drop table `%s`;", table))
		return alter
	}
```

### 代码更新后打包
> go build -o mysql-schema-sync main.go

### 修改python代码里的路径
```
json_path = "/Users/taojiaju/Work/code/mysql-schema-sync/config.json"
mysqlsync = "/Users/taojiaju/Work/code/mysql-schema-sync/mysql-schema-sync"
```

### 参考资料
> https://github.com/hidu/mysql-schema-sync

> https://www.geek-share.com/detail/2754929593.html

> https://blog.csdn.net/liyyzz33/article/details/84587300

> https://www.cnblogs.com/Camiluo/p/13359532.html

> https://www.jianshu.com/p/22f4b7860f42



