通过将表或列名包含在映射文档中的反勾中，可以强制Hibernate在生成的SQL中引用标识符。不过在传统上，Hibernate使用反勾来转义SQL保留关键字，而JPA使用双引号。

一旦转义了保留的关键字，Hibernate将为SQL方言使用正确的引号样式。这通常是双引号，但是SQL Server使用括号，MySQL使用反勾号。

###### 例61.Hibernate遗留引用

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Long id;

    @Column(name = "`name`")
    private String name;

    @Column(name = "`number`")
    private String number;

    //Getters and setters are omitted for brevity

}
```

###### 例62.JPA引用

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Long id;

    @Column(name = "\"name\"")
    private String name;

    @Column(name = "\"number\"")
    private String number;

    //Getters and setters are omitted for brevity

}
```

因为`name`和`number`是保留字，所以`Product`实体映射使用反勾号来引用这些列名。

当保存以下`Product`实体时，Hibernate生成以下SQL插入语句：

###### 例63.持久化引用的列名

```java
Product product = new Product();
product.setId( 1L );
product.setName( "Mobile phone" );
product.setNumber( "123-456-7890" );
entityManager.persist( product );
```

```java
INSERT INTO Product ("name", "number", id)
VALUES ('Mobile phone', '123-456-7890', 1)
```


