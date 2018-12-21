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

```java
INSERT INTO Client (name, id)
VALUES ('John Doe', 1)

INSERT INTO Account (amount, client_id, rate, account_type, id)
VALUES (5000.0, 1, 0.0125, 'CREDIT', 1)

INSERT INTO Account (amount, client_id, rate, account_type, id)
VALUES (0.0, 1, 0.0105, 'DEBIT', 2)

INSERT INTO Account (amount, client_id, rate, account_type, id)
VALUES (250.0, 1, 0.0105, 'DEBIT', 3)

INSERT INTO Client_Account (Client_id, order_id, accounts_id)
VALUES (1, 0, 1)

INSERT INTO Client_Account (Client_id, order_id, accounts_id)
VALUES (1, 0, 1)

INSERT INTO Client_Account (Client_id, order_id, accounts_id)
VALUES (1, 1, 2)

INSERT INTO Client_Account (Client_id, order_id, accounts_id)
VALUES (1, 2, 3)
```

只有在当前运行的Hibernate`Session`上启用关联筛选器时，才能筛选集合。

###### 例99.在不启用筛选器的情况下，遍历映射为`@FilterJoinTable`的集合

```java
Client client = entityManager.find( Client.class, 1L );

assertEquals( 3, client.getAccounts().size());
```

```java
SELECT
    ca.Client_id as Client_i1_2_0_,
    ca.accounts_id as accounts2_2_0_,
    ca.order_id as order_id3_0_,
    a.id as id1_0_1_,
    a.amount as amount3_0_1_,
    a.rate as rate4_0_1_,
    a.account_type as account_5_0_1_
FROM
    Client_Account ca
INNER JOIN
    Account a
ON  ca.accounts_id=a.id
WHERE
    ca.Client_id = ?

-- binding parameter [1] as [BIGINT] - [1]
```



