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



