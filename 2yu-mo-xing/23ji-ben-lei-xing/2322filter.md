```
@Filter注解是使用自定义SQL标准筛选实体或集合的另一种方法。与@Where注解不同，@Filter允许在运行时参数化筛选子句。
```

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

###### 例93.查询不激活`@Filter`映射的实体

```java
List<Account> accounts = entityManager.createQuery(
    "select a from Account a", Account.class)
.getResultList();

assertEquals( 3, accounts.size());
```

```java
SELECT
    a.id as id1_0_,
    a.active_status as active2_0_,
    a.amount as amount3_0_,
    a.client_id as client_i6_0_,
    a.rate as rate4_0_,
    a.account_type as account_5_0_
FROM
    Account a
```

如果启用了过滤器并且提供了过滤器参数值，那么Hibernate将向关联的Account实体应用过滤标准。

###### 例94.查询使用@Filter映射的实体

```java
entityManager
    .unwrap( Session.class )
    .enableFilter( "activeAccount" )
    .setParameter( "active", true);

List<Account> accounts = entityManager.createQuery(
    "select a from Account a", Account.class)
.getResultList();

assertEquals( 2, accounts.size());
```

```java
SELECT
    a.id as id1_0_,
    a.active_status as active2_0_,
    a.amount as amount3_0_,
    a.client_id as client_i6_0_,
    a.rate as rate4_0_,
    a.account_type as account_5_0_
FROM
    Account a
WHERE
    a.active_status = true
```

> 过滤器应用到实体查询，但不直接获取。
>
> 因此，在下面的例子中，当从持久化上下文中获取一个实体时过滤器不纳入考虑。
>
> ###### 获取带有`@Filter`映射的实体
>
> ```java
> entityManager
>     .unwrap( Session.class )
>     .enableFilter( "activeAccount" )
>     .setParameter( "active", true);
>
> Account account = entityManager.find( Account.class, 2L );
>
> assertFalse( account.isActive() );
> ```
>
> ```java
> SELECT
>     a.id as id1_0_0_,
>     a.active_status as active2_0_0_,
>     a.amount as amount3_0_0_,
>     a.client_id as client_i6_0_0_,
>     a.rate as rate4_0_0_,
>     a.account_type as account_5_0_0_,
>     c.id as id1_1_1_,
>     c.name as name2_1_1_
> FROM
>     Account a
> WHERE
>     a.id = 2
> ```
>
> 正如从上面的例子所见，与实体查询相反，过滤器不阻止实体被加载。

就像实体查询一样，也可以过滤集合，但只有在当前运行的Hibernate`Session`上显式启用了过滤器时才能过滤集合。

###### 例95.在不激活`@Filter`的情况下遍历集合

```java
Client client = entityManager.find( Client.class, 1L );

assertEquals( 3, client.getAccounts().size() );
```

```java
SELECT
    c.id as id1_1_0_,
    c.name as name2_1_0_
FROM
    Client c
WHERE
    c.id = 1

SELECT
    a.id as id1_0_,
    a.active_status as active2_0_,
    a.amount as amount3_0_,
    a.client_id as client_i6_0_,
    a.rate as rate4_0_,
    a.account_type as account_5_0_
FROM
    Account a
WHERE
    a.client_id = 1
```

当激活`@Filter`并获取帐户集合时，Hibernate将向关联的集合条目应用筛选条件。

###### 例96.遍历使用`@Filter`映射的集合

```java
entityManager
    .unwrap( Session.class )
    .enableFilter( "activeAccount" )
    .setParameter( "active", true);

Client client = entityManager.find( Client.class, 1L );

assertEquals( 2, client.getAccounts().size() );
```

```java
SELECT
    c.id as id1_1_0_,
    c.name as name2_1_0_
FROM
    Client c
WHERE
    c.id = 1

SELECT
    a.id as id1_0_,
    a.active_status as active2_0_,
    a.amount as amount3_0_,
    a.client_id as client_i6_0_,
    a.rate as rate4_0_,
    a.account_type as account_5_0_
FROM
    Account a
WHERE
    accounts0_.active_status = true
    and a.client_id = 1
```

> `@Filter`优于`@Where`子句的主要优点是可以在运行时自定义筛选条件。

> 无法结合@Filter和@Cache集合注释。此限制是由于确保一致性，因为过滤信息不存储在第二级缓存中。
>
> 如果当前筛选的集合允许缓存，那么第二级缓存将只存储整个集合的子集。之后，即使会话级别的过滤器没有被显式激活，其他每个Session都将从缓存中获取经过筛选的集合。
>
> 由于这个原因，第二级集合缓存仅限于存储整个集合，而不是子集。



