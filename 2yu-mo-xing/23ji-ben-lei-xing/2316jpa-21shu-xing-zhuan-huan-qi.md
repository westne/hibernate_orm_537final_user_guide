尽管Hibernate长期以来一直提供[自定义类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-custom-type)，但是作为JPA 2.1提供者，它也支持`AttributeConverter`。

使用自定义`AttributeConverter`，应用程序开发人员可以将给定的JDBC类型映射到实体基本类型。

在下面的示例中，`java.time.Period`将被映射到`VARCHAR`数据库列。

###### 例58.`java.time.Period`自定义`AttributeConverter`

```java
@Converter
public class PeriodStringConverter
        implements AttributeConverter<Period, String> {

    @Override
    public String convertToDatabaseColumn(Period attribute) {
        return attribute.toString();
    }

    @Override
    public Period convertToEntityAttribute(String dbData) {
        return Period.parse( dbData );
    }
}
```

要使用这个定制的转换器，@Convert注解必须修饰实体属性。

###### 例59.使用自定义的`java.time.Period`的`AttributeConverter`映射实体

```java
@Entity(name = "Event")
public static class Event {

    @Id
    @GeneratedValue
    private Long id;

    @Convert(converter = PeriodStringConverter.class)
    @Column(columnDefinition = "")
    private Period span;

    //Getters and setters are omitted for brevity

}
```

当持久化这样的实体时，Hibernate将根据`AttributeConverter`逻辑进行类型转换：

###### 例60.使用自定义`AttributeConverter`实现实体的持久化

```java
INSERT INTO Event ( span, id )
VALUES ( 'P1Y2M3D', 1 )
```

#### `AttributeConverter` Java和JDBC类型

在为转换的“数据库侧”指定的Java类型（第二个AttributeConverter绑定参数）不知道的情况下，Hibernate将回落到java.io.Serializable类型。



