Hibernate支持Java枚举类的映射作为基本值类型，以许多不同的方式。

##### `@Enumerated`

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

##### AttributeConvertor

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

###### 示例24.Enum映射与`AttributeConverter`示例

```java
@Entity(name = "Person")
public static class Person {

    @Id
    private Long id;

    private String name;

    @Convert( converter = GenderConverter.class )
    public Gender gender;

    //Getters and setters are omitted for brevity

}

@Converter
public static class GenderConverter
        implements AttributeConverter<Gender, Character> {

    public Character convertToDatabaseColumn( Gender value ) {
        if ( value == null ) {
            return null;
        }

        return value.getCode();
    }

    public Gender convertToEntityAttribute( Character value ) {
        if ( value == null ) {
            return null;
        }

        return Gender.fromCode( value );
    }
}
```

这里，gender列被定义为CHAR类型，并将保存：

###### `NULL`

空值

###### `“M”`

男性枚举

###### `“F”`

女性枚举

有关使用`AttributeConverters`的详细信息，请参阅[JPA 2.1属性转换器](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-jpa-convert)部分。

> JPA明确地禁止使用带有标记为`@Enumerated`的属性的AttributeConverter。因此，如果使用AttributeConverter方法，请确保不要将属性标记为`@Enumerated`。

##### 使用AttributeConverter实体属性作为查询参数

假设您具有以下实体：

###### 示例25.带有属性转换器的Photo实体

```java
@Entity(name = "Photo")
public static class Photo {

    @Id
    private Integer id;

    private String name;

    @Convert(converter = CaptionConverter.class)
    private Caption caption;

    //Getters and setters are omitted for brevity
}
```

`Caption`类如下所示：

###### 示例26.`Caption`Java对象

```java
public static class Caption {

    private String text;

    public Caption(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    @Override
    public boolean equals(Object o) {
        if ( this == o ) {
            return true;
        }
        if ( o == null || getClass() != o.getClass() ) {
            return false;
        }
        Caption caption = (Caption) o;
        return text != null ? text.equals( caption.text ) : caption.text == null;

    }

    @Override
    public int hashCode() {
        return text != null ? text.hashCode() : 0;
    }
}
```

并且我们有一个`AttributeConverter`来处理`Caption`Java对象：

###### 示例27.`Caption`Java对象属性转换器

```java
public static class CaptionConverter
        implements AttributeConverter<Caption, String> {

    @Override
    public String convertToDatabaseColumn(Caption attribute) {
        return attribute.getText();
    }

    @Override
    public Caption convertToEntityAttribute(String dbData) {
        return new Caption( dbData );
    }
}
```

传统上，在引用Caption实体属性时，只能使用dbData Caption表示，在本例中是String。

###### 示例28.使用DB数据表示通过Caption属性进行筛选

```java
Photo photo = entityManager.createQuery(
    "select p " +
    "from Photo p " +
    "where upper(caption) = upper(:caption) ", Photo.class )
.setParameter( "caption", "Nicolae Grigorescu" )
.getSingleResult();
```

为了使用Java对象`Caption`表示，必须获得相关的Hibernate`Type`。

###### 示例29.使用Java对象表示的`Caption`属性过滤

```java
SessionFactory sessionFactory = entityManager.getEntityManagerFactory()
        .unwrap( SessionFactory.class );

MetamodelImplementor metamodelImplementor = (MetamodelImplementor) sessionFactory.getMetamodel();

Type captionType = metamodelImplementor
        .entityPersister( Photo.class.getName() )
        .getPropertyType( "caption" );

Photo photo = (Photo) entityManager.createQuery(
    "select p " +
    "from Photo p " +
    "where upper(caption) = upper(:caption) ", Photo.class )
.unwrap( Query.class )
.setParameter( "caption", new Caption("Nicolae Grigorescu"), captionType)
.getSingleResult();
```

通过传递相关联的Hibernate`Type`，您可以在绑定查询参数值时使用`Caption`对象。

##### 使用HBM映射映射属性转换器

当使用HBM映射时，您仍然可以使用JPA`AttributeConverter`，因为Hibernate通过类型属性支持这种映射，如下面的示例所示。

让我们考虑我们有一个应用程序特定的`Money`类型：

###### 示例30.特定于应用程序的`Money`类型

```java
public class Money {

    private long cents;

    public Money(long cents) {
        this.cents = cents;
    }

    public long getCents() {
        return cents;
    }

    public void setCents(long cents) {
        this.cents = cents;
    }
}
```

现在，我们希望在映射Account实体时使用Money类型：

###### 示例31.使用Money类型的Account实体

```java
public class Account {

    private Long id;

    private String owner;

    private Money balance;

    //Getters and setters are omitted for brevity
}
```

由于Hibernate不知道如何持久化Money类型，所以我们可以使用JPA AttributeConverter将Money类型转换为Long。为此，我们将使用以下MoneyConverter工具：

###### 示例32.实现JPA`AttributeConverter`接口的MoneyConverter

```java
public class MoneyConverter
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

要使用HBM配置文件映射`MoneyConverter`，需要在property元素的`type`属性中使用`converted::`前缀。

###### 示例33.`AttributeConverter`的HBM映射

```java
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="org.hibernate.userguide.mapping.converter.hbm">
    <class name="Account" table="account" >
        <id name="id"/>

        <property name="owner"/>

        <property name="balance"
            type="converted::org.hibernate.userguide.mapping.converter.hbm.MoneyConverter"/>

    </class>
</hibernate-mapping>
```

##### 自定义类型

您还可以使用Hibernate自定义类型映射来映射枚举。让我们再次访问Gender枚举示例，这次使用自定义类型存储更标准化的“M”和“F”代码。

###### 示例34.使用自定义类型的枚举映射示例

```java
@Entity(name = "Person")
public static class Person {

    @Id
    private Long id;

    private String name;

    @Type( type = "org.hibernate.userguide.mapping.basic.GenderType" )
    public Gender gender;

    //Getters and setters are omitted for brevity

}

public class GenderType extends AbstractSingleColumnStandardBasicType<Gender> {

    public static final GenderType INSTANCE = new GenderType();

    public GenderType() {
        super(
            CharTypeDescriptor.INSTANCE,
            GenderJavaTypeDescriptor.INSTANCE
        );
    }

    public String getName() {
        return "gender";
    }

    @Override
    protected boolean registerUnderJavaType() {
        return true;
    }
}

public class GenderJavaTypeDescriptor extends AbstractTypeDescriptor<Gender> {

    public static final GenderJavaTypeDescriptor INSTANCE =
        new GenderJavaTypeDescriptor();

    protected GenderJavaTypeDescriptor() {
        super( Gender.class );
    }

    public String toString(Gender value) {
        return value == null ? null : value.name();
    }

    public Gender fromString(String string) {
        return string == null ? null : Gender.valueOf( string );
    }

    public <X> X unwrap(Gender value, Class<X> type, WrapperOptions options) {
        return CharacterTypeDescriptor.INSTANCE.unwrap(
            value == null ? null : value.getCode(),
            type,
            options
        );
    }

    public <X> Gender wrap(X value, WrapperOptions options) {
        return Gender.fromCode(
            CharacterTypeDescriptor.INSTANCE.wrap( value, options )
        );
    }
}
```

同样，gender列被定义为CHAR类型，并将保存：

###### `NULL`

空值

###### `“M”`

男性枚举

###### `“F”`

女性枚举

有关使用自定义类型的其他详细信息，请参阅[自定义BasicTypes](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-custom-type)部分。

