JDBC 4添加了显式处理国家化字符数据的能力。为此，它添加了特定的国家化字符数据类型：

* `NCHAR`

* `NVARCHAR`

* `LONGNVARCHAR`

* `NCLOB`

考虑到我们有以下数据库表：

###### 例46.`NVARCHAR`-SQL

```java
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

与`CLOB`一样，Hibernate也可以处理`NCLOB`SQL数据类型：

###### 例48.`NCLOB`- SQL

```java
CREATE TABLE Product (
    id INTEGER NOT NULL ,
    name VARCHAR(255) ,
    warranty nclob ,
    PRIMARY KEY ( id )
)
```

Hibernate可以将`NCLOB`映射到`java.sql.NClob`

###### 例49.映射到`java.sql.NClob`的`NCLOB`

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    private String name;

    @Lob
    @Nationalized
    // Clob also works, because NClob extends Clob.
    // The database type is still NCLOB either way and handled as such.
    private NClob warranty;

    //Getters and setters are omitted for brevity

}
```

要持久化这样的实体，必须使用`NClobProxy`Hibernate实用程序创建`NClob`：

###### 例50.持久化`java.sql.NClob`实体

```java
String warranty = "My product warranty";

final Product product = new Product();
product.setId( 1 );
product.setName( "Mobile phone" );

product.setWarranty( NClobProxy.generateProxy( warranty ) );

entityManager.persist( product );
```

要检索`NClob`内容，需要转换底层`java.io.Reader`：

###### 例51.返回`java.sql.NClob`实体

```java
Product product = entityManager.find( Product.class, productId );

try (Reader reader = product.getWarranty().getCharacterStream()) {
    assertEquals( "My product warranty", toString( reader ) );
}
```

我们还可以以物化的形式映射`NCLOB`。这样，我们可以使用`String`或`char[]`。

###### 例52.映射到`String`的`NCLOB`

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    private String name;

    @Lob
    @Nationalized
    private String warranty;

    //Getters and setters are omitted for brevity

}
```

我们甚至可能希望将物化数据作为char数组。

###### 例53.`NCLOB`- 物化`char[]`映射

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    private String name;

    @Lob
    @Nationalized
    private char[] warranty;

    //Getters and setters are omitted for brevity

}
```



