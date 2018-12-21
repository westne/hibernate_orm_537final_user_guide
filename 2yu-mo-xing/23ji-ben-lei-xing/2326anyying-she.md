还有一种类型的属性映射。@Any映射定义了与来自多个表的类的多态关联。这种类型的映射需要多于一列。第一列包含关联实体的类型。其余列包含标识符。

> 为这种关联指定外键约束是不可能的。这不是映射多态关联的常用方法，您应该只在特殊情况下（例如，审计日志、用户会话数据等）使用这种方法。

`@Any`注解描述了保存元数据信息的列。为了链接元数据信息的值和实际的实体类型，使用`@AnyDef`和`@AnyDefs`注解。`metaType`属性允许应用程序指定一个自定义类型，该类型将数据库列值映射到具有`idType`所指定类型的标识符属性的持久类。必须指定从`metaType`的值到类名的映射。

对于下面一个示例，考虑以下`Property`类层次结构：

###### 例103.`Property`类层次结构

```java
public interface Property<T> {

    String getName();

    T getValue();
}


@Entity
@Table(name="integer_property")
public class IntegerProperty implements Property<Integer> {

    @Id
    private Long id;

    @Column(name = "`name`")
    private String name;

    @Column(name = "`value`")
    private Integer value;

    @Override
    public String getName() {
        return name;
    }

    @Override
    public Integer getValue() {
        return value;
    }

    //Getters and setters omitted for brevity
}


@Entity
@Table(name="string_property")
public class StringProperty implements Property<String> {

    @Id
    private Long id;

    @Column(name = "`name`")
    private String name;

    @Column(name = "`value`")
    private String value;

    @Override
    public String getName() {
        return name;
    }

    @Override
    public String getValue() {
        return value;
    }

    //Getters and setters omitted for brevity
}
```



