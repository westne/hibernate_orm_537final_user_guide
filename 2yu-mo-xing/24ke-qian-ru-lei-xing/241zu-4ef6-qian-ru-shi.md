大多数情况下，可嵌入类型用于对多个基本类型映射进行分组，并在多个实体中重用它们。

###### 例124.简单可嵌入的

```java
@Entity(name = "Book")
public static class Book {

	@Id
	@GeneratedValue
	private Long id;

	private String title;

	private String author;

	private Publisher publisher;

	//Getters and setters are omitted for brevity
}

@Embeddable
public static class Publisher {

	@Column(name = "publisher_name")
	private String name;

	@Column(name = "publisher_country")
	private String country;

	//Getters and setters, equals and hashCode methods omitted for brevity

}
```

```ruby
create table Book (
    id bigint not null,
    author varchar(255),
    publisher_country varchar(255),
    publisher_name varchar(255),
    title varchar(255),
    primary key (id)
)
```

> JPA定义了两个用于处理可嵌入类型的术语：`@Embeddable`和`@Embedded`。
>
> `@Embeddable`用于描述映射类型本身（例如，`Publisher`）。
>
> `@Embedded`用于引用给定的可嵌入类型（例如，`book#publisher`）。

因此，可嵌入类型由`Publisher`类表示，父实体通过`book#publisher`对象组合使用它。

组合的值被映射到与父表相同的表。组合是良好的面向对象数据建模（惯用Java）的一部分。事实上，该表也可以由以下实体类型来映射。

###### 例125.可嵌入型组合的替代品

```java
@Entity(name = "Book")
public static class Book {

	@Id
	@GeneratedValue
	private Long id;

	private String title;

	private String author;

	@Column(name = "publisher_name")
	private String publisherName;

	@Column(name = "publisher_country")
	private String publisherCountry;

	//Getters and setters are omitted for brevity
}
```

组合形式当然更面向对象，并且随着我们处理多个可嵌入类型而变得更加明显。

