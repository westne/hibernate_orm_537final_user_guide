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

假设我们持久化了以下`City`实体：

###### 例121.`@Parent`持久化示例

```java
doInJPA( this::entityManagerFactory, entityManager -> {

	City cluj = new City();
	cluj.setName( "Cluj" );
	cluj.setCoordinates( new GPS( 46.77120, 23.62360 ) );

	entityManager.persist( cluj );
} );
```



