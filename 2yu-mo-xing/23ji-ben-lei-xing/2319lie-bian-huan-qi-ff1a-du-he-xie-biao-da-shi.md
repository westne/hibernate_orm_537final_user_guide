Hibernate允许您定制用于读取和写入映射到`@Basic`类型的列的值的SQL。例如，如果您的数据库提供了一组数据加密函数，那么您可以对单个列调用它们，如下面的示例所示。

###### 例78.@ColumnTransformer示例

```java
@Entity(name = "Employee")
public static class Employee {

	@Id
	private Long id;

	@NaturalId
	private String username;

	@Column(name = "pswd")
	@ColumnTransformer(
		read = "decrypt( 'AES', '00', pswd  )",
		write = "encrypt('AES', '00', ?)"
	)
	private String password;

	private int accessLevel;

	@ManyToOne(fetch = FetchType.LAZY)
	private Department department;

	@ManyToMany(mappedBy = "employees")
	private List<Project> projects = new ArrayList<>();

	//Getters and setters omitted for brevity
}
```

> 如果超过一个列需要定义这些规则中的任何一个，可以使用复数形式@ColumnTransformers。

如果属性使用多于一列，则必须使用`forColumn`属性指定表达式的目标列。

###### 例79.`@ColumnTransformer` `forColumn`属性使用

```java
@Entity(name = "Savings")
public static class Savings {

	@Id
	private Long id;

	@Type(type = "org.hibernate.userguide.mapping.basic.MonetaryAmountUserType")
	@Columns(columns = {
		@Column(name = "money"),
		@Column(name = "currency")
	})
	@ColumnTransformer(
		forColumn = "money",
		read = "money / 100",
		write = "? * 100"
	)
	private MonetaryAmount wallet;

	//Getters and setters omitted for brevity

}
```



