Strictly speaking, a basic type is denoted by the`javax.persistence.Basic`annotation. Generally speaking, the`@Basic`annotation can be ignored, as it is assumed by default. Both of the following examples are ultimately the same.

###### 示例3.`@Basic`显式声明

```java
@Entity(name = "Product")
public class Product {

    @Id
    @Basic
    private Integer id;

    @Basic
    private String sku;

    @Basic
    private String name;

    @Basic
    private String description;
}
```

###### 示例4.`@Basic`隐式声明

```java
@Entity(name = "Product")
public class Product {

    @Id
    private Integer id;

    private String sku;

    private String name;

    private String description;
}
```

> JPA规范严格限制了可以被标记为basic的Java类型到以下列表：
>
> * Java原语类型（布尔、int等）
>
> * 原语类型的包装器（java.lang.Boolean、java.lang.Integer等）
>
> * java.lang.String
>
> * java.math.BigInteger
>
> * java.math.BigDecimal
>
> * java.util.Date
>
> * java.util.Calendar
>
> * java.sql.Date
> * java.sql.Time
> * java.sql.Timestamp
> * byte\[\] or Byte\[\]
> * char\[\] or Character\[\]
> * enums
>
> * 实现Serializable的任何其他类型（JPA对Serializable类型的“支持”是直接将它们的状态序列化到数据库）。
>
> 如果需要考虑提供程序的可移植性，那么应该只使用这些基本类型。请注意，JPA 2.1确实添加了`javax.persistence.AttributeConverter`的概念，以帮助减轻其中一些顾虑；有关此主题的更多信息，请参阅JPA [2.1AttributeConverters](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-jpa-convert)。

`@Basic`定义了两个属性。

`optional` - boolean（缺省为true）

定义此属性是否允许空。JPA将此定义为“提示”，这实质上意味着它特别需要效果。只要类型不是基本类型，Hibernate就认为这意味着潜在的列应该是NULLABLE。

`fetch` - FetchType（缺省为EAGER）

定义是否应急切地或惰性地获取此属性。JPA说，EAGER是提供者（Hibernate）的一个要求，即当获取所有者时应该获取值，而LAZY仅仅是在访问属性时获取值的提示。Hibernate忽略基本类型的此设置，除非您正在使用字节码增强。有关获取和字节码增强的附加信息，请参阅[BytecodeEnhancement](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#BytecodeEnhancement)。

