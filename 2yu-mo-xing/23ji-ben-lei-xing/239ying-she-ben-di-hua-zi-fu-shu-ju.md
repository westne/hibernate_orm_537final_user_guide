JDBC 4添加了显式处理国家化字符数据的能力。为此，它添加了特定的国家化字符数据类型：

* `NCHAR`

* `NVARCHAR`

* `LONGNVARCHAR`

* `NCLOB`

考虑到我们有以下数据库表：

###### 例46.`NVARCHAR`-SQL

```SQL
CREATE TABLE Product (
    id INTEGER NOT NULL ,
    name VARCHAR(255) ,
    warranty NVARCHAR(255) ,
    PRIMARY KEY ( id )
)
```



