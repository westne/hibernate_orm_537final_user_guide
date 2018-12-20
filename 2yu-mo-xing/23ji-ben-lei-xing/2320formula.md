有时，您希望数据库为您执行一些计算，而不是在JVM中执行一些计算，还可以创建某种虚拟列。可以使用SQL片段（也称为公式）而不是将属性映射到列中。此类属性是只读的（其值由公式片段计算）

> 您应该知道，@Formula注解使用了可能会影响数据库可移植性的本地SQL子句。

###### 例81.`@Formula`映射使用

```java
@Entity(name = "Account")
public static class Account {

    @Id
    private Long id;

    private Double credit;

    private Double rate;

    @Formula(value = "credit * rate")
    private Double interest;

    //Getters and setters omitted for brevity

}
```

在加载`Account`实体时，Hibernate将使用配置的`@Formula`：

###### 例82.持久化具有`@Formula`映射的实体

```java
doInJPA( this::entityManagerFactory, entityManager -> {
	Account account = new Account( );
	account.setId( 1L );
	account.setCredit( 5000d );
	account.setRate( 1.25 / 100 );
	entityManager.persist( account );
} );

doInJPA( this::entityManagerFactory, entityManager -> {
	Account account = entityManager.find( Account.class, 1L );
	assertEquals( Double.valueOf( 62.5d ), account.getInterest());
} );
```

```java
INSERT INTO Account (credit, rate, id)
VALUES (5000.0, 0.0125, 1)

SELECT
    a.id as id1_0_0_,
    a.credit as credit2_0_0_,
    a.rate as rate3_0_0_,
    a.credit * a.rate as formula0_0_
FROM
    Account a
WHERE
    a.id = 1
```



