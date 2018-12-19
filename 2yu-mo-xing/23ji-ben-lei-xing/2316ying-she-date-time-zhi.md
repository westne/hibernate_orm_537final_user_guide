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

> 与`java.util.Date`一样，`java.util.Calendar`需要`@Temporal`注释，以便知道要选择什么JDBC数据类型：`DATE`、`TIME`或`TIMESTAMP`。如果`java.util.Date`标记了一个时间点，那么`java.util.Calendar`将考虑默认时区。



