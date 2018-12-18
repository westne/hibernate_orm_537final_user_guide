有时您希望以不同的方式处理特定的属性。偶尔Hibernate会隐式地选择您不想要的`BasicType`（由于某些原因，您不想调整`BasicTypeRegistry`）。

在这些情况下，必须通过`org.hibernate.annotations.Type`注释显式地告诉Hibernate要使用的`BasicType`。

###### 示例6.使用`@org.hibernate.annotations.Type`

```java
@Entity(name = "Product")
public class Product {

    @Id
    private Integer id;

    private String sku;

    @org.hibernate.annotations.Type( type = "nstring" )
    private String name;

    @org.hibernate.annotations.Type( type = "materialized_nclob" )
    private String description;
}
```

这告诉Hibernate将String存储为本地化数据。这只是为了说明的目的；为了更好地指示本地化字符数据，请参阅[映射本地化字符数据](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-nationalized)部分。

此外，description将作为LOB处理。同样，有关指示LOB的更好方法，请参阅[映射LOB](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-lob)部分。

`org.hibernate.annotations.Type#type`属性可以命名以下任何属性：

* 任何`org.hibernate.type.Type`实现的完全限定名称

* 在`BasicTypeRegistry`注册的任何密钥

* 任何已知_类型定义_的名称



