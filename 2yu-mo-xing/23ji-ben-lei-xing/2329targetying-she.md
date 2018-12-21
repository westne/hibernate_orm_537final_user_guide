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

坐标可嵌入类型被映射为坐标接口。但是，Hibernate需要知道实际的实现类型，在本例中是GPS，因此@Target注解用于提供此信息。

假设我们持久化以下`City`实体：

###### 例118.`@Target`持久化示例

```java
doInJPA( this::entityManagerFactory, entityManager -> {

	City cluj = new City();
	cluj.setName( "Cluj" );
	cluj.setCoordinates( new GPS( 46.77120, 23.62360 ) );

	entityManager.persist( cluj );
} );
```



