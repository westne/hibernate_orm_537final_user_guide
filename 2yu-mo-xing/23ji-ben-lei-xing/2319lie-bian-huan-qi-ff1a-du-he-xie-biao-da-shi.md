Hibernate允许您定制用于读取和写入映射到`@Basic`类型的列的值的SQL。例如，如果您的数据库提供了一组数据加密函数，那么您可以对单个列调用它们，如下面的示例所示。

###### 例78.@ColumnTransformer示例

```java
@Entity(name = "Employee")
public static class Employee {

    @Id
    private Long id;

    @NaturalId
    private String username;

    @Column(name = "pswd")
    @ColumnTransformer(
        read = "decrypt( 'AES', '00', pswd  )",
        write = "encrypt('AES', '00', ?)"
    )
    private String password;

    private int accessLevel;

    @ManyToOne(fetch = FetchType.LAZY)
    private Department department;

    @ManyToMany(mappedBy = "employees")
    private List<Project> projects = new ArrayList<>();

    //Getters and setters omitted for brevity
}
```

> 如果超过一个列需要定义这些规则中的任何一个，可以使用复数形式@ColumnTransformers。

如果属性使用多于一列，则必须使用`forColumn`属性指定表达式的目标列。

###### 例79.`@ColumnTransformer` `forColumn`属性使用

```java
@Entity(name = "Savings")
public static class Savings {

    @Id
    private Long id;

    @Type(type = "org.hibernate.userguide.mapping.basic.MonetaryAmountUserType")
    @Columns(columns = {
        @Column(name = "money"),
        @Column(name = "currency")
    })
    @ColumnTransformer(
        forColumn = "money",
        read = "money / 100",
        write = "? * 100"
    )
    private MonetaryAmount wallet;

    //Getters and setters omitted for brevity

}
```

Hibernate在查询中引用属性时自动应用自定义表达式。此功能类似于派生属性`@Formula`，有两个区别：

* 属性由一个或多个列支持，这些列作为自动模式生成的一部分导出。
* 属性是读写，而不是只读。

如果指定了写表达式，则必须恰好包含一个“?”值的占位符。

###### 例80.持久化一个使用`@ColumnTransformer`和复合类型的实体

```java
doInJPA( this::entityManagerFactory, entityManager -> {
	Savings savings = new Savings( );
	savings.setId( 1L );
	savings.setWallet( new MonetaryAmount( BigDecimal.TEN, Currency.getInstance( Locale.US ) ) );
	entityManager.persist( savings );
} );

doInJPA( this::entityManagerFactory, entityManager -> {
	Savings savings = entityManager.find( Savings.class, 1L );
	assertEquals( 10, savings.getWallet().getAmount().intValue());
} );
```

```java
INSERT INTO Savings (money, currency, id)
VALUES (10 * 100, 'USD', 1)

SELECT
    s.id as id1_0_0_,
    s.money / 100 as money2_0_0_,
    s.currency as currency3_0_0_
FROM
    Savings s
WHERE
    s.id = 1
```



