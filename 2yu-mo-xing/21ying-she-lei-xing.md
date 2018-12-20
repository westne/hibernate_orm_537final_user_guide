Hibernate同时理解应用程序数据的Java和JDBC表示。从数据库读取/写入这些数据的能力是Hibernate_类型_的功能。在这种用法中，类型是`org.hibernate.type.Type`接口的实现。Hibernate类型还描述了Java类型的各种行为方面，例如如何检查相等性、如何克隆值等。

> _类型的用法_
>
> Hibernate类型既不是Java类型，也不是SQL数据类型。它提供了关于这两个方面的信息以及理解它们之间的编组\(marshalling\)。
>
> 当遇到Hibernate讨论中的术语类型时，它可能指代Java类型、JDBC类型或Hibernate类型，根据上下文。

为了帮助理解类型分类，让我们看一个我们希望映射的简单表和域模型。

###### 示例1.一个简单的表和域模型

```SQL
create table Contact(
    id integer not null,
    first varchar(255),
    last varchar(255),
    middle varchar(255),
    notes varchar(255),
    starred boolean not null,
    website varchar(255),
    primary key (id)
)
```

```java
@Entity(name = "Contact")
public static class Contact {

 @Id
 private Integer id;
 private Name name;
 private String notes;
 private URL website;
 private boolean starred;
 //Getters and setters are omitted for brevity
}

@Embeddable
public class Name {
 private String first;
 private String middle;
 private String last;
 // getters and setters omitted
}
```

在最广泛的意义上，Hibernate将类型分为两组：

* [值类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#categorization-value)
* [实体类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#categorization-entity)



