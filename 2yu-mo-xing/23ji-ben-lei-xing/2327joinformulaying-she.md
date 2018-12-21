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

```java
CREATE TABLE countries (
    id int4 NOT NULL,
    name VARCHAR(255),
    PRIMARY KEY ( id )
)

CREATE TABLE users (
    id int8 NOT NULL,
    firstName VARCHAR(255),
    lastName VARCHAR(255),
    phoneNumber VARCHAR(255),
    PRIMARY KEY ( id )
)
```

`User`实体中的国家关联由`phoneNumber`属性提供的国家标识符映射。

考虑到我们有以下实体：

###### 例112.`@JoinFormula`映射使用

```java
Country US = new Country();
US.setId( 1 );
US.setName( "United States" );

Country Romania = new Country();
Romania.setId( 40 );
Romania.setName( "Romania" );

doInJPA( this::entityManagerFactory, entityManager -> {
    entityManager.persist( US );
    entityManager.persist( Romania );
} );

doInJPA( this::entityManagerFactory, entityManager -> {
    User user1 = new User( );
    user1.setId( 1L );
    user1.setFirstName( "John" );
    user1.setLastName( "Doe" );
    user1.setPhoneNumber( "+1-234-5678" );
    entityManager.persist( user1 );

    User user2 = new User( );
    user2.setId( 2L );
    user2.setFirstName( "Vlad" );
    user2.setLastName( "Mihalcea" );
    user2.setPhoneNumber( "+40-123-4567" );
    entityManager.persist( user2 );
} );
```

当获取`User`实体时，`country`属性由`@JoinFormula`表达式映射：

###### 例113.`@JoinFormula`映射使用





