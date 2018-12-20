有时，您希望数据库为您执行一些计算，而不是在JVM中执行一些计算，还可以创建某种虚拟列。可以使用SQL片段（也称为公式）而不是将属性映射到列中。此类属性是只读的（其值由公式片段计算）

> 您应该知道，@Formula注解使用了可能会影响数据库可移植性的本地SQL子句。

###### 例81.`@Formula`映射使用

```java
@Entity(name = "Account")
public static class Account {

	@Id
	private Long id;

	private Double credit;

	private Double rate;

	@Formula(value = "credit * rate")
	private Double interest;

	//Getters and setters omitted for brevity

}
```



