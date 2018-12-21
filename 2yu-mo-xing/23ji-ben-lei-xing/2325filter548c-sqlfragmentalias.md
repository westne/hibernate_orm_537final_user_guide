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



