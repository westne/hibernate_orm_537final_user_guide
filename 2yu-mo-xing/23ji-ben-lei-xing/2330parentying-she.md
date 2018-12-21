Hibernate特定的`@Parent`注解允许您从可嵌入类型中引用所有者实体。

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

在获取`City`实体时，可嵌入类型的`city`属性充当对拥有者父实体的反向引用：

###### 例122.`@Parent`获取示例

```java
doInJPA( this::entityManagerFactory, entityManager -> {

	City cluj = entityManager.find( City.class, 1L );

	assertSame( cluj, cluj.getCoordinates().getCity() );
} );
```



