`@Filter`注解是使用自定义SQL标准筛选实体或集合的另一种方法。与@Where注解不同，@Filter允许在运行时参数化筛选子句。

现在，考虑到我们有以下Account实体：

###### 例90.`@Filter`映射实体级使用

```java
@Entity(name = "Account")
@FilterDef(
    name="activeAccount",
    parameters = @ParamDef(
        name="active",
        type="boolean"
    )
)
@Filter(
    name="activeAccount",
    condition="active_status = :active"
)
public static class Account {

    @Id
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Client client;

    @Column(name = "account_type")
    @Enumerated(EnumType.STRING)
    private AccountType type;

    private Double amount;

    private Double rate;

    @Column(name = "active_status")
    private boolean active;

    //Getters and setters omitted for brevity
}
```

> 注意，`active`属性被映射到`active_status`列。
>
> 此映射是为了显示`@Filter`条件使用SQL条件而不是JPQL筛选谓词。

如前所述，我们还可以对集合应用`@Filter`注释，如`Client`实体所示：

###### 例91.`@Filter`映射集合级使用

```java
@Entity(name = "Client")
public static class Client {

    @Id
    private Long id;

    private String name;

    @OneToMany(
        mappedBy = "client",
        cascade = CascadeType.ALL
    )
    @Filter(
        name="activeAccount",
        condition="active_status = :active"
    )
    private List<Account> accounts = new ArrayList<>( );

    //Getters and setters omitted for brevity

    public void addAccount(Account account) {
        account.setClient( this );
        this.accounts.add( account );
    }
}
```

如果我们使用三个相关联的`Account`实体持久化一个`Client`，Hibernate将执行以下SQL语句：

###### 例92.使用`@Filter`映射持久化和获取实体

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
    .setActive( true )
);

client.addAccount(
    new Account()
    .setId( 2L )
    .setType( AccountType.DEBIT )
    .setAmount( 0d )
    .setRate( 1.05 / 100 )
    .setActive( false )
);

client.addAccount(
    new Account()
    .setType( AccountType.DEBIT )
    .setId( 3L )
    .setAmount( 250d )
    .setRate( 1.05 / 100 )
    .setActive( true )
);

entityManager.persist( client );
```

```java
INSERT INTO Client (name, id)
VALUES ('John Doe', 1)

INSERT INTO Account (active_status, amount, client_id, rate, account_type, id)
VALUES (true, 5000.0, 1, 0.0125, 'CREDIT', 1)

INSERT INTO Account (active_status, amount, client_id, rate, account_type, id)
VALUES (false, 0.0, 1, 0.0105, 'DEBIT', 2)

INSERT INTO Account (active_status, amount, client_id, rate, account_type, id)
VALUES (true, 250.0, 1, 0.0105, 'DEBIT', 3)
```

默认情况下，在不显式启用筛选器的情况下，Hibernate将获取所有`Account`实体。

###### 例93.在不激活`@Filter`的情况下查询映射的实体

```java
List<Account> accounts = entityManager.createQuery(
    "select a from Account a", Account.class)
.getResultList();

assertEquals( 3, accounts.size());
```



