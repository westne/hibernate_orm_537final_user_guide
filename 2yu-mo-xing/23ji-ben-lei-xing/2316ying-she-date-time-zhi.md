Hibernate支持各种Java Date/Time类作为持久域模型实体属性被映射。SQL标准定义了三种Date/Time类型：

`DATE`

代表一个日历日期，存储年月日。 JDBC等价是`java.sql.Date`

`TIME`

代表一天中的时间，存储时分秒。JDBC等价是`java.sql.Time`

`TIMESTAMP`

存储一个DATE和一个TIME加纳秒。JDBC等价是`java.sql.Timestamp`

> 为了避免对`java.sql`包的依赖，通常使用`java.util`或`java.time`包的Date/Time类。

当`java.sql`类定义与SQL Date/Time数据类型的直接关联时，`java.util`或`java.time`属性需要显式地将SQL类型关联标记为`@Temporal` 注解。通过这种方式，可以将`java.util.Date`或`java.util.Calendar`映射到SQL `DATE`、`TIME`或`TIMESTAMP`类型。

考虑以下实体：

###### 例54.`java.util.Date`映射为`DATE`

```java
@Entity(name = "DateEvent")
public static class DateEvent {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "`timestamp`")
    @Temporal(TemporalType.DATE)
    private Date timestamp;

    //Getters and setters are omitted for brevity

}
```

当持久化这样的实体时：

###### 例55.持久化`java.util.Date`映射

```java
DateEvent dateEvent = new DateEvent( new Date() );
entityManager.persist( dateEvent );
```

Hibernate生成以下INSERT语句：

```java
INSERT INTO DateEvent ( timestamp, id )
VALUES ( '2015-12-29', 1 )
```

只有年、月和日字段被保存到数据库中。

如果我们将`@Temporal`类型更改为`TIME`：

###### 例56.`java.util.Date`映射为`TIME`

```java
@Column(name = "`timestamp`")
@Temporal(TemporalType.TIME)
private Date timestamp;
```

Hibernate将发出包含小时、分钟和秒的INSERT语句。

```java
INSERT INTO DateEvent ( timestamp, id )
VALUES ( '16:51:58', 1 )
```

当`@Temporal`类型设置为`TIMESTAMP`时：

###### 例57.`java.util.Date`映射为`TIMESTAMP`

```java
@Column(name = "`timestamp`")
@Temporal(TemporalType.TIMESTAMP)
private Date timestamp;
```

Hibernate将在INSERT语句中包括DATE、TIME和纳秒：

```java
INSERT INTO DateEvent ( timestamp, id )
VALUES ( '2015-12-29 16:54:04.544', 1 )
```

> 与`java.util.Date`一样，`java.util.Calendar`需要`@Temporal`注释，以便知道要选择什么JDBC数据类型：DATE、TIME或TIMESTAMP。如果`java.util.Date`标记了一个时间点，那么`java.util.Calendar`将考虑默认时区。

#### 映射Java 8 Date/Time值

Java 8附带了一个新的Date/Time API，它支持Java.Times包中的即时日期、间隔、本地和分区Date/Time不可变实例。

标准SQLDate/Time类型与所支持的Java 8 Date/Time类类型之间的映射如下；

###### `DATE`

`java.time.LocalDate`

###### `TIME`

`java.time.LocalTime`,`java.time.OffsetTime`

###### `TIMESTAMP`

`java.time.Instant`,`java.time.LocalDateTime`,`java.time.OffsetDateTime`和`java.time.ZonedDateTime`

> 因为Java 8日期/时间类与SQL类型之间的映射是隐式的，所以不需要指定`@Temporal`注释。将其设置在`java.times`类上会引发以下异常：
>
> `org.hibernate.AnnotationException: @Temporal should only be set on a java.util.Date or java.util.Calendar property`

#### 使用特定时区

默认情况下，Hibernate在保存java.sql.Timestamp或者java.sql.Time属性时，将使用[PreparedStatement.setTimestamp\(int parameterIndex, java.sql.Timestamp\)](https://docs.oracle.com/javase/8/docs/api/java/sql/PreparedStatement.html#setTimestamp-int-java.sql.Timestamp-)或者[PreparedStatement.setTime\(int parameterIndex, java.sql.Time x\)](https://docs.oracle.com/javase/8/docs/api/java/sql/PreparedStatement.html#setTime-int-java.sql.Time-)。

当没有指定时区时，JDBC驱动程序将使用底层的JVM默认时区，如果应用程序来自全球各地，这可能不适合。由于这个原因，每当从数据库保存/加载数据时，使用单个参考时区（例如，UTC）是非常常见的。

一种替代方法是配置所有JVM以使用参考时区：

###### 声明地

```java
java -Duser.timezone=UTC ...
```

###### 以编程方式

```java
TimeZone.setDefault( TimeZone.getTimeZone( "UTC" ) );
```

然而，正如[本文](http://in.relation.to/2016/09/12/jdbc-time-zone-configuration-property/)中所解释的，这并不总是实用的，尤其是对于前端节点。为此，Hibernate提供了`hibernate.jdbc.time_zone`配置属性，该属性可以被配置：

###### 声明性地，在SessionFactory级别

```java
settings.put(
    AvailableSettings.JDBC_TIME_ZONE,
    TimeZone.getTimeZone( "UTC" )
);
```



