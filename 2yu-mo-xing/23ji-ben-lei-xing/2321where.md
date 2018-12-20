有时，您希望使用自定义的SQL criteria筛选出实体或集合。这可以通过使用@Where注解来实现，它可以应用于实体和集合。

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



