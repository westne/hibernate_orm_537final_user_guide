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

##### `AttributeConverter` Java和JDBC类型

在转换的“数据库侧”所指定的Java类型（第二个`AttributeConverter`绑定参数）不知道的情况下，Hibernate将回落到`java.io.Serializable`类型。

如果Java类型是Hibernate不知道的，您将遇到以下消息：

-- HHH000481:遭遇到某种Java类型，对它我们无法找到JavaTypeDescriptor，它似乎没有实现等值和/或哈希代码。当执行涉及Java类型的相等/脏检查时，这会导致显著的性能问题。考虑注册一个自定义的JavaTypeDescriptor或者至少实现equals/hashCode。

Java类型是否为“已知”意味着它在JavaTypeDescriptorRegistry中有一个条目。虽然默认情况下，Hibernate将许多JDK类型加载到JavaTypeDescriptorRegistry，但是应用程序也可以通过添加新的JavaTypeDescriptor条目来扩展JavaTypeDescriptorRegistry。

这样，Hibernate也将知道如何在JDBC级别处理特定的Java对象类型。

##### JPA 2.1`AttributeConverter`易变性计划

一个被JPA`AttributeConverter`转变的基本类型是不可变的，如果底层的Java类型不可变，反之如果关联的属性类型可变则也可变。

因此，关联的实体属性类型的[`JavaTypeDescriptor#getMutabilityPlan`](https://docs.jboss.org/hibernate/orm/5.3/javadocs/org/hibernate/type/descriptor/java/JavaTypeDescriptor.html#getMutabilityPlan--)提供了可变性。

##### 不可变类型

如果实体属性是`String`、基本类型包装类（例如，`Integer`、`Long`）、Enum类型或任何其他不可变`Object`类型，则只能通过将实体属性值重新分配到新值来更改实体属性值。

考虑到我们有与[JPA 2.1AttributeConverters](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-jpa-convert)部分中所示的相同的Period实体属性：

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

改变span属性的唯一方法是将其重新分配到不同的值：

```java
Event event = entityManager.createQuery( "from Event", Event.class ).getSingleResult();
event.setSpan(Period
    .ofYears( 3 )
    .plusMonths( 2 )
    .plusDays( 1 )
);
```

##### 可变类型

另一方面，考虑以下示例，其中Money类型是可变的。

```java
public static class Money {

    private long cents;

    //Getters and setters are omitted for brevity
}

@Entity(name = "Account")
public static class Account {

    @Id
    private Long id;

    private String owner;

    @Convert(converter = MoneyConverter.class)
    private Money balance;

    //Getters and setters are omitted for brevity
}

public static class MoneyConverter
        implements AttributeConverter<Money, Long> {

    @Override
    public Long convertToDatabaseColumn(Money attribute) {
        return attribute == null ? null : attribute.getCents();
    }

    @Override
    public Money convertToEntityAttribute(Long dbData) {
        return dbData == null ? null : new Money( dbData );
    }
}
```

可变对象允许您修改其内部结构，Hibernate脏检查机制将把更改传播到数据库：

```java
Account account = entityManager.find( Account.class, 1L );
account.getBalance().setCents( 150 * 100L );
entityManager.persist( account );
```

> 尽管`AttributeConverter`类型可以是可变的，因此脏检查、深度复制和第二级缓存可以正常工作，但是将这些类型视为不可变的（当它们确实是时）更有效。
>
> 由于这个原因，尽可能地选择不可变类型而不是可变类型。



