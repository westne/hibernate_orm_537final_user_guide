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

`PropertyHolder`可以引用任何此类属性，并且由于每个`Property`都属于一个单独的表，因此需要`@Any`注解。

###### 例104.`@Any`映射使用

```java
@Entity
@Table( name = "property_holder" )
public class PropertyHolder {

    @Id
    private Long id;

    @Any(
        metaDef = "PropertyMetaDef",
        metaColumn = @Column( name = "property_type" )
    )
    @JoinColumn( name = "property_id" )
    private Property property;

    //Getters and setters are omitted for brevity

}
```

```java
CREATE TABLE property_holder (
    id BIGINT NOT NULL,
    property_type VARCHAR(255),
    property_id BIGINT,
    PRIMARY KEY ( id )
)
```

如您所见，有两列用于引用`Property`实例：`property_id`和`property_type`。`property_id`用于匹配`string_property`或`integer_property`表的`id`列，而`property_type`用于匹配`string_property`或`integer_property`表。

表解析映射由`metaDef`属性定义，该属性引用`@AnyMetaDef`映射。虽然`@AnyMetaDef`映射可以紧挨`@Any`注解设置，但是重用它是一个很好的实践，因此在类或包级别配置它是有意义的。

`package-info.java`包含`@AnyMetaDef`映射：

###### 例105.`@Any`映射使用

```java
@AnyMetaDef( name= "PropertyMetaDef", metaType = "string", idType = "long",
    metaValues = {
            @MetaValue(value = "S", targetEntity = StringProperty.class),
            @MetaValue(value = "I", targetEntity = IntegerProperty.class)
        }
    )
package org.hibernate.userguide.mapping.basic.any;

import org.hibernate.annotations.AnyMetaDef;
import org.hibernate.annotations.MetaValue;
```

> 建议将`@AnyMetaDef`映射作为包元数据放置。

要查看`@Any`注解的实际效果，请考虑下面的示例。

如果我们持久化`IntegerProperty`以及`StringProperty`实体，并将`StringProperty`实体与`PropertyHolder`关联，Hibernate将生成以下SQL查询：

###### 例106.`@Any`映射持久化示例

```java
IntegerProperty ageProperty = new IntegerProperty();
ageProperty.setId( 1L );
ageProperty.setName( "age" );
ageProperty.setValue( 23 );

session.persist( ageProperty );

StringProperty nameProperty = new StringProperty();
nameProperty.setId( 1L );
nameProperty.setName( "name" );
nameProperty.setValue( "John Doe" );

session.persist( nameProperty );

PropertyHolder namePropertyHolder = new PropertyHolder();
namePropertyHolder.setId( 1L );
namePropertyHolder.setProperty( nameProperty );

session.persist( namePropertyHolder );
```

```java
INSERT INTO integer_property
       ( "name", "value", id )
VALUES ( 'age', 23, 1 )

INSERT INTO string_property
       ( "name", "value", id )
VALUES ( 'name', 'John Doe', 1 )

INSERT INTO property_holder
       ( property_type, property_id, id )
VALUES ( 'S', 1, 1 )
```

当获取`PropertyHolder`实体并导航其属性关联时，Hibernate将获取关联的`StringProperty`实体，如下所示：

###### 例107.`@Any`映射查询示例

```java
PropertyHolder propertyHolder = session.get( PropertyHolder.class, 1L );

assertEquals("name", propertyHolder.getProperty().getName());
assertEquals("John Doe", propertyHolder.getProperty().getValue());
```

```java
SELECT ph.id AS id1_1_0_,
       ph.property_type AS property2_1_0_,
       ph.property_id AS property3_1_0_
FROM   property_holder ph
WHERE  ph.id = 1


SELECT sp.id AS id1_2_0_,
       sp."name" AS name2_2_0_,
       sp."value" AS value3_2_0_
FROM   string_property sp
WHERE  sp.id = 1
```

##### `@ManyToAny`映射 {#mapping-column-many-to-any}

当存在多个目标实体时，`@Any`映射对于模拟`@ManyToOne`关联很有用。为了模拟`@OneToMany`关联，必须使用`@ManyToAny`注解。

在下面的示例中，`PropertyRepository`实体具有`Property`实体的集合。

`repository_properties`链接表保存`PropertyRepository`和`Property`实体之间的关联。

###### 例108.`@ManyToAny`映射使用

```java
@Entity
@Table( name = "property_repository" )
public class PropertyRepository {

    @Id
    private Long id;

    @ManyToAny(
        metaDef = "PropertyMetaDef",
        metaColumn = @Column( name = "property_type" )
    )
    @Cascade( { org.hibernate.annotations.CascadeType.ALL })
    @JoinTable(name = "repository_properties",
        joinColumns = @JoinColumn(name = "repository_id"),
        inverseJoinColumns = @JoinColumn(name = "property_id")
    )
    private List<Property<?>> properties = new ArrayList<>(  );

    //Getters and setters are omitted for brevity

}
```

```java
CREATE TABLE property_repository (
    id BIGINT NOT NULL,
    PRIMARY KEY ( id )
)

CREATE TABLE repository_properties (
    repository_id BIGINT NOT NULL,
    property_type VARCHAR(255),
    property_id BIGINT NOT NULL
)
```



