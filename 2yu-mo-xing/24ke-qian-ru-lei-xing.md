在历史上，Hibernate调用这些组件。JPA称之为可嵌入的。无论哪种方式，概念都是一样的：值的组合。

例如，我们可能有一个Publisher类，它是name和country的组合，或者一个Location类，它是country和city的组合。

> ###### embeddable一词的用法
>
> 为了避免与标记给定可嵌入类型的注解混淆，注解将进一步称为`@Embeddable`。
>
> 为了简洁起见，在本章及其后的整个章节中，可嵌入类型也可以称为_可嵌入_。

###### 例123.可嵌入类型示例

```java
@Embeddable
public static class Publisher {

    private String name;

    private Location location;

    public Publisher(String name, Location location) {
        this.name = name;
        this.location = location;
    }

    private Publisher() {}

    //Getters and setters are omitted for brevity
}

@Embeddable
public static class Location {

    private String country;

    private String city;

    public Location(String country, String city) {
        this.country = country;
        this.city = city;
    }

    private Location() {}

    //Getters and setters are omitted for brevity
}
```

可嵌入类型是值类型的另一种形式，其生命周期绑定到父实体类型，因此从父实体类型继承属性访问（有关属性访问的详细信息，请参阅[访问策略](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#access-embeddable-types)）。

可嵌入类型可以由基本值以及关联组成，但需要注意的是，当用作集合元素时，它们不能定义集合本身。

