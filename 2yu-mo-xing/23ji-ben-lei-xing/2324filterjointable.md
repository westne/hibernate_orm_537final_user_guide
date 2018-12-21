在集合中使用`@Filter`注释时，对子条目（实体或可嵌入项）进行过滤。但是，如果在父实体和子表之间有一个链接表，则需要使用`@FilterJoinTable`根据联接表中包含的某个列过滤子条目。

因此，`@FilterJoinTable`注释可以应用于单向`@OneToMany`集合，如下面的映射所示：

###### 例97.`@FilterJoinTable`映射使用

```java
@Entity(name = "Client")
@FilterDef(
    name="firstAccounts",
    parameters=@ParamDef(
        name="maxOrderId",
        type="int"
    )
)
@Filter(
    name="firstAccounts",
    condition="order_id <= :maxOrderId"
)
public static class Client {

    @Id
    private Long id;

    private String name;

    @OneToMany(cascade = CascadeType.ALL)
    @OrderColumn(name = "order_id")
    @FilterJoinTable(
        name="firstAccounts",
        condition="order_id <= :maxOrderId"
    )
    private List<Account> accounts = new ArrayList<>( );

    //Getters and setters omitted for brevity

    public void addAccount(Account account) {
        this.accounts.add( account );
    }
}

@Entity(name = "Account")
public static class Account {

    @Id
    private Long id;

    @Column(name = "account_type")
    @Enumerated(EnumType.STRING)
    private AccountType type;

    private Double amount;

    private Double rate;

    //Getters and setters omitted for brevity
}
```

`firstAccounts`过滤器将允许我们仅获得`order_id`（它告诉帐户集合内的每个条目的位置）小于给定数字（例如，`maxOrderId`）的`Account`实体。

假设我们的数据库包含以下实体：

###### 例98.持久化和获取使用`@FilterJoinTable`映射的实体

```java
Client client = new Client()
.setId( 1L )
.setName( "John Doe" );

client.addAccount(
    new Account()
    .setId( 1L )
    .setType( AccountType.CREDIT )
    .setAmount( 5000d )
    .setRate( 1.25 / 100 )
);

client.addAccount(
    new Account()
    .setId( 2L )
    .setType( AccountType.DEBIT )
    .setAmount( 0d )
    .setRate( 1.05 / 100 )
);

client.addAccount(
    new Account()
    .setType( AccountType.DEBIT )
    .setId( 3L )
    .setAmount( 250d )
    .setRate( 1.05 / 100 )
);

entityManager.persist( client );
```



