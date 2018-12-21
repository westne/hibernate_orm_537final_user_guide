`@Target`注解用于指定通过接口映射的给定关联的实现类。`@ManyToOne`、`@OneToOne`、`@OneToMany`、`@ManyToMany`具有一个`targetEntity`属性，以便在使用接口进行映射时指定实体关联的实际类。

`@ElementCollection`关联具有用于相同目的的`targetClass`属性。

然而，对于简单的可嵌入类型，没有这样的构造，因此您需要使用Hibernate特定的`@Target`注解。

###### 例117.`@Target`映射使用

```java
public interface Coordinates {
	double x();
	double y();
}

@Embeddable
public static class GPS implements Coordinates {

	private double latitude;

	private double longitude;

	private GPS() {
	}

	public GPS(double latitude, double longitude) {
		this.latitude = latitude;
		this.longitude = longitude;
	}

	@Override
	public double x() {
		return latitude;
	}

	@Override
	public double y() {
		return longitude;
	}
}

@Entity(name = "City")
public static class City {

	@Id
	@GeneratedValue
	private Long id;

	private String name;

	@Embedded
	@Target( GPS.class )
	private Coordinates coordinates;

	//Getters and setters omitted for brevity

}
```



