```
有时，您希望使用自定义的SQL criteria筛选出实体或集合。这可以通过使用@Where注解来实现，它可以应用于实体和集合。
```

###### 例83.`@where`映射使用

```java
public enum AccountType {
    DEBIT,
    CREDIT
}

@Entity(name = "Client")
public static class Client {

    @Id
    private Long id;

    private String name;

    @Where( clause = "account_type = 'DEBIT'")
    @OneToMany(mappedBy = "client")
    private List<Account> debitAccounts = new ArrayList<>( );

    @Where( clause = "account_type = 'CREDIT'")
    @OneToMany(mappedBy = "client")
    private List<Account> creditAccounts = new ArrayList<>( );

    //Getters and setters omitted for brevity

}

@Entity(name = "Account")
@Where( clause = "active = true" )
public static class Account {

    @Id
    private Long id;

    @ManyToOne
    private Client client;

    @Column(name = "account_type")
    @Enumerated(EnumType.STRING)
    private AccountType type;

    private Double amount;

    private Double rate;

    private boolean active;

    //Getters and setters omitted for brevity

}
```

如果数据库包含以下实体：

###### 例84.持久化和获取使用`@Where`映射的实体

```java
doInJPA( this::entityManagerFactory, entityManager -> {

    Client client = new Client();
    client.setId( 1L );
    client.setName( "John Doe" );
    entityManager.persist( client );

    Account account1 = new Account( );
    account1.setId( 1L );
    account1.setType( AccountType.CREDIT );
    account1.setAmount( 5000d );
    account1.setRate( 1.25 / 100 );
    account1.setActive( true );
    account1.setClient( client );
    client.getCreditAccounts().add( account1 );
    entityManager.persist( account1 );

    Account account2 = new Account( );
    account2.setId( 2L );
    account2.setType( AccountType.DEBIT );
    account2.setAmount( 0d );
    account2.setRate( 1.05 / 100 );
    account2.setActive( false );
    account2.setClient( client );
    client.getDebitAccounts().add( account2 );
    entityManager.persist( account2 );

    Account account3 = new Account( );
    account3.setType( AccountType.DEBIT );
    account3.setId( 3L );
    account3.setAmount( 250d );
    account3.setRate( 1.05 / 100 );
    account3.setActive( true );
    account3.setClient( client );
    client.getDebitAccounts().add( account3 );
    entityManager.persist( account3 );
} );
```

```java
INSERT INTO Client (name, id)
VALUES ('John Doe', 1)

INSERT INTO Account (active, amount, client_id, rate, account_type, id)
VALUES (true, 5000.0, 1, 0.0125, 'CREDIT', 1)

INSERT INTO Account (active, amount, client_id, rate, account_type, id)
VALUES (false, 0.0, 1, 0.0105, 'DEBIT', 2)

INSERT INTO Account (active, amount, client_id, rate, account_type, id)
VALUES (true, 250.0, 1, 0.0105, 'DEBIT', 3)
```

在执行`Account`实体查询时，Hibernate将过滤掉所有不活动的记录。

###### 例85.查询使用`@Where`映射的实体

```java
doInJPA( this::entityManagerFactory, entityManager -> {
	List<Account> accounts = entityManager.createQuery(
		"select a from Account a", Account.class)
	.getResultList();
	assertEquals( 2, accounts.size());
} );
```



