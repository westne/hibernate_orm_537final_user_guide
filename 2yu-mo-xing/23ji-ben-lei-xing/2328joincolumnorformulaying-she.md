`@JoinColumnOrFormula`注解用于在需要考虑列值以及`@JoinFormula`时自定义子外键和父行主键之间的连接。

###### 例114.`@JoinColumnOrFormula`映射使用

```java
@Entity(name = "User")
@Table(name = "users")
public static class User {

    @Id
    private Long id;

    private String firstName;

    private String lastName;

    private String language;

    @ManyToOne
    @JoinColumnOrFormula( column =
        @JoinColumn(
            name = "language",
            referencedColumnName = "primaryLanguage",
            insertable = false,
            updatable = false
        )
    )
    @JoinColumnOrFormula( formula =
        @JoinFormula(
            value = "true",
            referencedColumnName = "is_default"
        )
    )
    private Country country;

    //Getters and setters omitted for brevity

}

@Entity(name = "Country")
@Table(name = "countries")
public static class Country implements Serializable {

    @Id
    private Integer id;

    private String name;

    private String primaryLanguage;

    @Column(name = "is_default")
    private boolean _default;

    //Getters and setters, equals and hashCode methods omitted for brevity

}
```

```ruby
CREATE TABLE countries (
    id INTEGER NOT NULL,
    is_default boolean,
    name VARCHAR(255),
    primaryLanguage VARCHAR(255),
    PRIMARY KEY ( id )
)

CREATE TABLE users (
    id BIGINT NOT NULL,
    firstName VARCHAR(255),
    language VARCHAR(255),
    lastName VARCHAR(255),
    PRIMARY KEY ( id )
)
```

`User`实体中的`country`关联由`language`属性值和关联的`Country` `is_default`列值映射。

考虑到我们有以下实体：

###### 例115.`@JoinColumnOrFormula`持久化示例

```java
Country US = new Country();
US.setId( 1 );
US.setDefault( true );
US.setPrimaryLanguage( "English" );
US.setName( "United States" );

Country Romania = new Country();
Romania.setId( 40 );
Romania.setDefault( true );
Romania.setName( "Romania" );
Romania.setPrimaryLanguage( "Romanian" );

doInJPA( this::entityManagerFactory, entityManager -> {
    entityManager.persist( US );
    entityManager.persist( Romania );
} );

doInJPA( this::entityManagerFactory, entityManager -> {
    User user1 = new User( );
    user1.setId( 1L );
    user1.setFirstName( "John" );
    user1.setLastName( "Doe" );
    user1.setLanguage( "English" );
    entityManager.persist( user1 );

    User user2 = new User( );
    user2.setId( 2L );
    user2.setFirstName( "Vlad" );
    user2.setLastName( "Mihalcea" );
    user2.setLanguage( "Romanian" );
    entityManager.persist( user2 );

} );
```

当获取`User`实体时，`country`属性由`@JoinColumnOrFormula`表达式映射：

###### 例116.`@JoinColumnOrFormula`获取示例

```java
doInJPA( this::entityManagerFactory, entityManager -> {
	log.info( "Fetch User entities" );

	User john = entityManager.find( User.class, 1L );
	assertEquals( US, john.getCountry());

	User vlad = entityManager.find( User.class, 2L );
	assertEquals( Romania, vlad.getCountry());
} );
```

```ruby
SELECT
    u.id as id1_1_0_,
    u.language as language3_1_0_,
    u.firstName as firstNam2_1_0_,
    u.lastName as lastName4_1_0_,
    1 as formula1_0_,
    c.id as id1_0_1_,
    c.is_default as is_defau2_0_1_,
    c.name as name3_0_1_,
    c.primaryLanguage as primaryL4_0_1_
FROM
    users u
LEFT OUTER JOIN
    countries c
        ON u.language = c.primaryLanguage
        AND 1 = c.is_default
WHERE
    u.id = ?

-- binding parameter [1] as [BIGINT] - [1]

SELECT
    u.id as id1_1_0_,
    u.language as language3_1_0_,
    u.firstName as firstNam2_1_0_,
    u.lastName as lastName4_1_0_,
    1 as formula1_0_,
    c.id as id1_0_1_,
    c.is_default as is_defau2_0_1_,
    c.name as name3_0_1_,
    c.primaryLanguage as primaryL4_0_1_
FROM
    users u
LEFT OUTER JOIN
    countries c
        ON u.language = c.primaryLanguage
        AND 1 = c.is_default
WHERE
    u.id = ?

-- binding parameter [1] as [BIGINT] - [2]
```

因此，`@JoinColumnOrFormula`注解用于定义父子关联之间的自定义联接关联。

