`@JoinFormula`注释用于自定义子外键和父行主键之间的连接。

###### 例111.`@JoinFormula`映射使用

```java
@Entity(name = "User")
@Table(name = "users")
public static class User {

	@Id
	private Long id;

	private String firstName;

	private String lastName;

	private String phoneNumber;

	@ManyToOne
	@JoinFormula( "REGEXP_REPLACE(phoneNumber, '\\+(\\d+)-.*', '\\1')::int" )
	private Country country;

	//Getters and setters omitted for brevity

}

@Entity(name = "Country")
@Table(name = "countries")
public static class Country {

	@Id
	private Integer id;

	private String name;

	//Getters and setters, equals and hashCode methods omitted for brevity

}
```



