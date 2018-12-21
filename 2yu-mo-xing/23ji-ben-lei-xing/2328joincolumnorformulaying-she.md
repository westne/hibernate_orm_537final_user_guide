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

例115.`@JoinColumnOrFormula`持久化示例



