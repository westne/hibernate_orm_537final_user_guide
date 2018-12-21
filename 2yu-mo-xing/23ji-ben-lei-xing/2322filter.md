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

如前所述，我们还可以对集合应用`@Filter`注释，如客户端实体所示：

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



