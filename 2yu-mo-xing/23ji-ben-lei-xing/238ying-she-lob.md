映射LOB（数据库大对象）有两种形式，一种是使用JDBC定位器类型，另一种是物化LOB数据。

JDBC LOB定位器的存在是为了允许对LOB数据的有效访问。它们允许JDBC驱动程序根据需要流式传输部分LOB数据，从而可能释放内存空间。然而，它们可能是不自然的处理，并有一定的局限性。例如，LOB定位器仅在获得它的事务期间可移植地有效。

物化LOB的思想是权衡潜在的效率（不是所有的驱动程序都有效地处理LOB数据），对于更自然的编程范例，使用熟悉的Java类型，如String或byte\[\]等，用于这些LOB。

物化处理内存中的整个LOB内容，而LOB定位器（理论上）允许根据需要将LOB内容的一部分流式传输入内存。

JDBC LOB定位器类型包括：

* java.sql.Blob
* java.sql.Clob
* java.sql.NClob

映射这些LOB值的物化形式将使用更熟悉的Java类型，如String、char\[\]、byte\[\]等。对比较熟悉的折衷通常是性能。

#### 映射CLOB

首先，假设我们有一个希望映射的`CLOB`列（[映射本地化字符数据](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-nationalized)部分将讨论`NCLOB`字符`LOB`数据）。

考虑到我们有以下数据库表：

###### 示例35.Culb－SQL

```SQL
CREATE TABLE Product (
  id INTEGER NOT NULL,
  name VARCHAR(255),
  warranty CLOB,
  PRIMARY KEY (id)
)
```

让我们首先使用`@Lob`JPA注解和`java.sql.Clob`类型对此进行映射：

###### 示例36.`CLOB`映射到`java.sql.Clob`

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    private String name;

    @Lob
    private Clob warranty;

    //Getters and setters are omitted for brevity

}
```

要持久化这样的实体，必须使用`ClobProxy`Hibernate工具创建Clob：

###### 例37.持久化`java.sql.Clob`实体

```java
String warranty = "My product warranty";

final Product product = new Product();
product.setId( 1 );
product.setName( "Mobile phone" );

product.setWarranty( ClobProxy.generateProxy( warranty ) );

entityManager.persist( product );
```



