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

JPA规范严格限制了可以被标记为basic的Java类型到以下列表：

* Java原语类型（布尔、int等）

* 原语类型的包装器（java.lang.Boolean、java.lang.Integer等）
* java.lang.String
* java.math.BigInteger
* java.math.BigDecimal
* java.util.Date
* java.util.Calendar
* java.sql.Date
* java.sql.Time
* java.sql.Timestamp
* byte\[\] or Byte\[\]
* char\[\] or Character\[\]
* enums



