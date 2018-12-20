生成的属性是由数据库生成其值的属性。通常，Hibernate应用程序需要`refresh`某对象，该对象包含数据库为其生成值的任何属性。但是，将属性标记为被生成的，允许应用程序将此责任委托给Hibernate。当Hibernate为已定义生成属性的实体发出SQL INSERT或UPDATE时，它立即发出一个select来取回生成的值。

标记为被生成的属性还必须是_不可插入_和_不可更新_的。只有`@Version`和`@Basic`类型可以标记为被生成的。

`NEVER`\(默认\)

给出的属性值不由数据库内生成。

`INSERT`

给出的属性值在插入时被生成，但是子序列更新时不生成。属性如`creationTimestamp`归为这个分类。

`ALWAYS`

属性值在插入和更新时都生成。

若要将属性标记为被生成的，请使用Hibernate特定的`@Generated`注解。

#### `@Generated`注解

使用`@Generated`注解，以便在实体被持久化或更新之后，Hibernate可以获取当前已注解的属性。由于这个原因，`@Generated`注解接受[`GenerationTime`](https://docs.jboss.org/hibernate/orm/5.3/javadocs/org/hibernate/annotations/GenerationTime.html)枚举值。

考虑以下实体：

###### 例65.`@Generated`映射示例

```java
@Entity(name = "Person")
public static class Person {

    @Id
    private Long id;

    private String firstName;

    private String lastName;

    private String middleName1;

    private String middleName2;

    private String middleName3;

    private String middleName4;

    private String middleName5;

    @Generated( value = GenerationTime.ALWAYS )
    @Column(columnDefinition =
        "AS CONCAT(" +
        "    COALESCE(firstName, ''), " +
        "    COALESCE(' ' + middleName1, ''), " +
        "    COALESCE(' ' + middleName2, ''), " +
        "    COALESCE(' ' + middleName3, ''), " +
        "    COALESCE(' ' + middleName4, ''), " +
        "    COALESCE(' ' + middleName5, ''), " +
        "    COALESCE(' ' + lastName, '') " +
        ")")
    private String fullName;

}
```

当持久化`Person`实体时，Hibernate将从数据库获取计算的fullName列，该列连接first、middle和last姓名。

例66.`@Generated`持久化示例

