Hibernate支持Java枚举类的映射作为基本值类型，以许多不同的方式。

#### `@Enumerated`

最初的与JPA兼容的映射枚举的方法是通过`@Enumerate`或`@MapKeyEnumerated`作为映射键注释，其原理是枚举值根据javax.persistence.EnumType所指示的两种策略之一进行存储：

###### `ORDINAL`

根据枚举类中枚举值的顺序位置存储，如`Java.Lang.Enum#ordinal`所指示的。

###### `STRING`

按照枚举值的名称存储，如`java.lang.Enum#name`所指示的。

假设以下枚举：

```java
public enum PhoneType {
    LAND_LINE,
    MOBILE;
}
```

在ORDINAL示例中，`phone_type`列被定义为一个\(nullable\) INTEGER类型，并将保存：

`NULL`

对于 空值

`0`

对于`LAND_LINE`枚举

`1`

对于`MOBILE`枚举

###### 示例19.`@Enumerated(ORDINAL)`例子

```java
@Entity(name = "Phone")
public static class Phone {

    @Id
    private Long id;

    @Column(name = "phone_number")
    private String number;

    @Enumerated(EnumType.ORDINAL)
    @Column(name = "phone_type")
    private PhoneType type;

    //Getters and setters are omitted for brevity

}
```

在持久化这个实体类时，Hibernate生成以下语句：

```java
Phone phone = new Phone( );
phone.setId( 1L );
phone.setNumber( "123-456-78990" );
phone.setType( PhoneType.MOBILE );
entityManager.persist( phone );
INSERT INTO Phone (phone_number, phone_type, id)
VALUES ('123-456-78990', 2, 1)
```

在STRING示例中，`phone_type`列被定义为一个\(nullable\) VARCHAR类型，并将保存：

###### `NULL`

空值

###### `LAND_LINE`

对于`LAND_LINE`枚举

###### `MOBILE`

用于`MOBILE`枚举

###### 示例21.@Enumerated\(STRING\)例子

```java
@Entity(name = "Phone")
public static class Phone {

    @Id
    private Long id;

    @Column(name = "phone_number")
    private String number;

    @Enumerated(EnumType.STRING)
    @Column(name = "phone_type")
    private PhoneType type;

    //Getters and setters are omitted for brevity

}
```

持久化`@Enumerated(ORDINAL)`例子中相同的持久类时，Hibernate生成以下的SQL语句：

###### 示例22.持久化一个使用`@Enumerated(STRING)`映射的实体

```java
INSERT INTO Phone (phone_number, phone_type, id)
VALUES ('123-456-78990', 'MOBILE', 1)
```

#### 属性转换器

让我们考虑下面的Gender枚举，它使用“M”和“F”代码存储它的值。

###### 示例23.使用自定义构造函数枚举

```java
public enum Gender {

    MALE( 'M' ),
    FEMALE( 'F' );

    private final char code;

    Gender(char code) {
        this.code = code;
    }

    public static Gender fromCode(char code) {
        if ( code == 'M' || code == 'm' ) {
            return MALE;
        }
        if ( code == 'F' || code == 'f' ) {
            return FEMALE;
        }
        throw new UnsupportedOperationException(
            "The code " + code + " is not supported!"
        );
    }

    public char getCode() {
        return code;
    }
}
```

可以使用JPA 2.1属性转换器以符合JPA的方式映射枚举。

