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





