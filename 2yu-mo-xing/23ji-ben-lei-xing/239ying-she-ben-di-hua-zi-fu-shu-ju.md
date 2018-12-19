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

为了将特定属性映射到国家化的变体数据类型，Hibernate定义了@Nationalized注解。

###### 例47.`NVARCHAR`映射

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    private String name;

    @Nationalized
    private String warranty;

    //Getters and setters are omitted for brevity

}
```



