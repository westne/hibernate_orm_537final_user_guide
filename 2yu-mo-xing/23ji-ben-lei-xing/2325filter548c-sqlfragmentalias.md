当使用`@Filter`注解并处理映射到多个数据库表上的实体时，如果`@Filter`定义了跨多个表使用谓词的条件，则需要使用`@SqlFragmentAlias`注解。

###### 例101.`@SqlFragmentAlias`映射使用

```java
@Entity(name = "Account")
@Table(name = "account")
@SecondaryTable(
    name = "account_details"
)
@SQLDelete(
    sql = "UPDATE account_details SET deleted = true WHERE id = ? "
)
@FilterDef(
    name="activeAccount",
    parameters = @ParamDef(
        name="active",
        type="boolean"
    )
)
@Filter(
    name="activeAccount",
    condition="{a}.active = :active and {ad}.deleted = false",
    aliases = {
        @SqlFragmentAlias( alias = "a", table= "account"),
        @SqlFragmentAlias( alias = "ad", table= "account_details"),
    }
)
public static class Account {

    @Id
    private Long id;

    private Double amount;

    private Double rate;

    private boolean active;

    @Column(table = "account_details")
    private boolean deleted;

    //Getters and setters omitted for brevity

}
```

现在，当获取`Account`实体并激活过滤器时，Hibernate将向过滤器谓词应用正确的表别名：

###### 例102.获取用`@SqlFragmentAlias`筛选的集合

```java
entityManager
	.unwrap( Session.class )
	.enableFilter( "activeAccount" )
	.setParameter( "active", true);

List<Account> accounts = entityManager.createQuery(
	"select a from Account a", Account.class)
.getResultList();
```

```java
select
    filtersqlf0_.id as id1_0_,
    filtersqlf0_.active as active2_0_,
    filtersqlf0_.amount as amount3_0_,
    filtersqlf0_.rate as rate4_0_,
    filtersqlf0_1_.deleted as deleted1_1_
from
    account filtersqlf0_
left outer join
    account_details filtersqlf0_1_
        on filtersqlf0_.id=filtersqlf0_1_.id
where
    filtersqlf0_.active = ?
    and filtersqlf0_1_.deleted = false

-- binding parameter [1] as [BOOLEAN] - [true]
```



