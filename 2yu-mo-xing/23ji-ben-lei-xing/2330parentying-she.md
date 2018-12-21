Hibernate特定的`@Parent`注解允许您从可嵌入中引用所有者实体。

###### 例120.@Parent映射使用

```java
@Embeddable
public static class GPS {

	private double latitude;

	private double longitude;

	@Parent
	private City city;

	//Getters and setters omitted for brevity

}

@Entity(name = "City")
public static class City {

	@Id
	@GeneratedValue
	private Long id;

	private String name;

	@Embedded
	@Target( GPS.class )
	private GPS coordinates;

	//Getters and setters omitted for brevity

}
```



