Hibernate支持Java枚举类的映射作为基本值类型，以许多不同的方式。

#### `@Enumerated`

最初的与JPA兼容的映射枚举的方法是通过`@Enumerate`或`@MapKeyEnumerated`作为映射键注释，其原理是枚举值根据javax.persistence.EnumType所指示的两种策略之一进行存储：

###### `ORDINAL`

根据枚举类中枚举值的顺序位置存储，如`Java.Lang.Enum#ordinal`所指示的。

###### `STRING`

按照枚举值的名称存储，如`java.lang.Enum#name`所指示的。

假设以下枚举：

```java
public enum PhoneType {
    LAND_LINE,
    MOBILE;
}
```

在ORDINAL示例中，`phone_type`列被定义为一个\(nullable\) INTEGER类型，并将保存：

`NULL`

对于 空值

`0`

对于`LAND_LINE`枚举

`1`

对于`MOBILE`枚举

###### 示例19.`@Enumerated(ORDINAL)`例子

```java
@Entity(name = "Phone")
public static class Phone {

	@Id
	private Long id;

	@Column(name = "phone_number")
	private String number;

	@Enumerated(EnumType.ORDINAL)
	@Column(name = "phone_type")
	private PhoneType type;

	//Getters and setters are omitted for brevity

}
```



