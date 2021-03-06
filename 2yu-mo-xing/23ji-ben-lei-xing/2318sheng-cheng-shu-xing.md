生成的属性是由数据库生成其值的属性。通常，Hibernate应用程序需要`refresh`某对象，该对象包含数据库为其生成值的任何属性。但是，将属性标记为被生成的，允许应用程序将此责任委托给Hibernate。当Hibernate为已定义生成属性的实体发出SQL INSERT或UPDATE时，它立即发出一个select来取回生成的值。

标记为被生成的属性还必须是_不可插入_和_不可更新_的。只有`@Version`和`@Basic`类型可以标记为被生成的。

`NEVER`\(默认\)

给出的属性值不由数据库内生成。

`INSERT`

给出的属性值在插入时被生成，但是子序列更新时不生成。属性如`creationTimestamp`归为这个分类。

`ALWAYS`

属性值在插入和更新时都生成。

若要将属性标记为被生成的，请使用Hibernate特定的`@Generated`注解。

##### `@Generated`注解

使用`@Generated`注解，以便在实体被持久化或更新之后，Hibernate可以获取当前已注解的属性。由于这个原因，`@Generated`注解接受[`GenerationTime`](https://docs.jboss.org/hibernate/orm/5.3/javadocs/org/hibernate/annotations/GenerationTime.html)枚举值。

考虑以下实体：

###### 例65.`@Generated`映射示例

```java
@Entity(name = "Person")
public static class Person {

    @Id
    private Long id;

    private String firstName;

    private String lastName;

    private String middleName1;

    private String middleName2;

    private String middleName3;

    private String middleName4;

    private String middleName5;

    @Generated( value = GenerationTime.ALWAYS )
    @Column(columnDefinition =
        "AS CONCAT(" +
        "    COALESCE(firstName, ''), " +
        "    COALESCE(' ' + middleName1, ''), " +
        "    COALESCE(' ' + middleName2, ''), " +
        "    COALESCE(' ' + middleName3, ''), " +
        "    COALESCE(' ' + middleName4, ''), " +
        "    COALESCE(' ' + middleName5, ''), " +
        "    COALESCE(' ' + lastName, '') " +
        ")")
    private String fullName;

}
```

当持久化`Person`实体时，Hibernate将从数据库获取计算的fullName列，该列连接first、middle和last姓名。

###### 例66.`@Generated`持久化示例

```java
Person person = new Person();
person.setId( 1L );
person.setFirstName( "John" );
person.setMiddleName1( "Flávio" );
person.setMiddleName2( "André" );
person.setMiddleName3( "Frederico" );
person.setMiddleName4( "Rúben" );
person.setMiddleName5( "Artur" );
person.setLastName( "Doe" );

entityManager.persist( person );
entityManager.flush();

assertEquals("John Flávio André Frederico Rúben Artur Doe", person.getFullName());
```

```java
INSERT INTO Person
(
    firstName,
    lastName,
    middleName1,
    middleName2,
    middleName3,
    middleName4,
    middleName5,
    id
)
values
(?, ?, ?, ?, ?, ?, ?, ?)

-- binding parameter [1] as [VARCHAR] - [John]
-- binding parameter [2] as [VARCHAR] - [Doe]
-- binding parameter [3] as [VARCHAR] - [Flávio]
-- binding parameter [4] as [VARCHAR] - [André]
-- binding parameter [5] as [VARCHAR] - [Frederico]
-- binding parameter [6] as [VARCHAR] - [Rúben]
-- binding parameter [7] as [VARCHAR] - [Artur]
-- binding parameter [8] as [BIGINT]  - [1]

SELECT
    p.fullName as fullName3_0_
FROM
    Person p
WHERE
    p.id=?

-- binding parameter [1] as [BIGINT] - [1]
-- extracted value ([fullName3_0_] : [VARCHAR]) - [John Flávio André Frederico Rúben Artur Doe]
```

更新`Person`实体时也是如此。Hibernate将在修改实体之后从数据库中获取计算的`fullName`列。

###### 例67.`@Generated`更新示例

```java
Person person = entityManager.find( Person.class, 1L );
person.setLastName( "Doe Jr" );

entityManager.flush();
assertEquals("John Flávio André Frederico Rúben Artur Doe Jr", person.getFullName());
```

```java
UPDATE
    Person
SET
    firstName=?,
    lastName=?,
    middleName1=?,
    middleName2=?,
    middleName3=?,
    middleName4=?,
    middleName5=?
WHERE
    id=?

-- binding parameter [1] as [VARCHAR] - [John]
-- binding parameter [2] as [VARCHAR] - [Doe Jr]
-- binding parameter [3] as [VARCHAR] - [Flávio]
-- binding parameter [4] as [VARCHAR] - [André]
-- binding parameter [5] as [VARCHAR] - [Frederico]
-- binding parameter [6] as [VARCHAR] - [Rúben]
-- binding parameter [7] as [VARCHAR] - [Artur]
-- binding parameter [8] as [BIGINT]  - [1]

SELECT
    p.fullName as fullName3_0_
FROM
    Person p
WHERE
    p.id=?

-- binding parameter [1] as [BIGINT] - [1]
-- extracted value ([fullName3_0_] : [VARCHAR]) - [John Flávio André Frederico Rúben Artur Doe Jr]
```

##### `@GeneratorType`注解

使用`@GeneratorType`注解可以提供自定义生成器来设置当前注释的属性的值。

因此，`@GeneratorType`注解接受`GenerationTime`枚举值和自定义`ValueGenerator`类类型。

考虑以下实体：

###### 例68.`@GeneratorType`映射示例

```java
public static class CurrentUser {

    public static final CurrentUser INSTANCE = new CurrentUser();

    private static final ThreadLocal<String> storage = new ThreadLocal<>();

    public void logIn(String user) {
        storage.set( user );
    }

    public void logOut() {
        storage.remove();
    }

    public String get() {
        return storage.get();
    }
}

public static class LoggedUserGenerator implements ValueGenerator<String> {

    @Override
    public String generateValue(
            Session session, Object owner) {
        return CurrentUser.INSTANCE.get();
    }
}

@Entity(name = "Person")
public static class Person {

    @Id
    private Long id;

    private String firstName;

    private String lastName;

    @GeneratorType( type = LoggedUserGenerator.class, when = GenerationTime.INSERT)
    private String createdBy;

    @GeneratorType( type = LoggedUserGenerator.class, when = GenerationTime.ALWAYS)
    private String updatedBy;

}
```

当持久化Person实体时，Hibernate将使用当前记录的用户填充`createdBy`列。

###### 例69.`@Generated`持久化示例

```java
CurrentUser.INSTANCE.logIn( "Alice" );

doInJPA( this::entityManagerFactory, entityManager -> {

    Person person = new Person();
    person.setId( 1L );
    person.setFirstName( "John" );
    person.setLastName( "Doe" );

    entityManager.persist( person );
} );

CurrentUser.INSTANCE.logOut();
```

```java
INSERT INTO Person
(
    createdBy,
    firstName,
    lastName,
    updatedBy,
    id
)
VALUES
(?, ?, ?, ?, ?)

-- binding parameter [1] as [VARCHAR] - [Alice]
-- binding parameter [2] as [VARCHAR] - [John]
-- binding parameter [3] as [VARCHAR] - [Doe]
-- binding parameter [4] as [VARCHAR] - [Alice]
-- binding parameter [5] as [BIGINT]  - [1]
```

更新Person实体时也是如此。Hibernate将使用当前记录的用户填充`updatedBy`列。

###### 例70.`@Generated`更新示例

```java
CurrentUser.INSTANCE.logIn( "Bob" );

doInJPA( this::entityManagerFactory, entityManager -> {
    Person person = entityManager.find( Person.class, 1L );
    person.setFirstName( "Mr. John" );
} );

CurrentUser.INSTANCE.logOut();
```

```java
UPDATE Person
SET
    createdBy = ?,
    firstName = ?,
    lastName = ?,
    updatedBy = ?
WHERE
    id = ?

-- binding parameter [1] as [VARCHAR] - [Alice]
-- binding parameter [2] as [VARCHAR] - [Mr. John]
-- binding parameter [3] as [VARCHAR] - [Doe]
-- binding parameter [4] as [VARCHAR] - [Bob]
-- binding parameter [5] as [BIGINT]  - [1]
```

##### `@CreationTimestamp`注解 {#mapping-generated-CreationTimestamp}

`@CreationTimestamp`注解指示Hibernate在持久化实体时使用JVM的当前时间戳值设置带注解的实体属性。

支持的属性类型是：

* `java.util.Date`

* `java.util.Calendar`

* `java.sql.Date`

* `java.sql.Time`

* `java.sql.Timestamp`

###### 例71.`@CreationTimestamp`映射示例

```java
@Entity(name = "Event")
public static class Event {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "`timestamp`")
    @CreationTimestamp
    private Date timestamp;

    //Constructors, getters, and setters are omitted for brevity
}
```

当持久化Event实体时，Hibernate将使用当前JVM时间戳值填充底层`timestamp`列：

###### 例72.`@CreationTimestamp`持久化示例

```java
Event dateEvent = new Event( );
entityManager.persist( dateEvent );
```

```java
INSERT INTO Event ("timestamp", id)
VALUES (?, ?)

-- binding parameter [1] as [TIMESTAMP] - [Tue Nov 15 16:24:20 EET 2016]
-- binding parameter [2] as [BIGINT]    - [1]
```

##### `@UpdateTimestamp`注解 {#mapping-generated-UpdateTimestamp}

`@UpdateTimestamp`注解指示Hibernate在持久化实体时使用JVM的当前时间戳值设置带注解的实体属性。

支持的属性类型是：

* `java.util.Date`

* `java.util.Calendar`

* `java.sql.Date`

* `java.sql.Time`

* `java.sql.Timestamp`

###### 例73.`@UpdateTimestamp`映射示例

```java
@Entity(name = "Bid")
public static class Bid {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "updated_on")
    @UpdateTimestamp
    private Date updatedOn;

    @Column(name = "updated_by")
    private String updatedBy;

    private Long cents;

    //Getters and setters are omitted for brevity

}
```

当持久化Bid实体时，Hibernate将使用当前JVM时间戳值填充底层的`update_on`列：

###### 例74.`@UpdateTimestamp`持久化示例

```java
Bid bid = new Bid();
bid.setUpdatedBy( "John Doe" );
bid.setCents( 150 * 100L );
entityManager.persist( bid );
```

```java
INSERT INTO Bid (cents, updated_by, updated_on, id)
VALUES (?, ?, ?, ?)

-- binding parameter [1] as [BIGINT]    - [15000]
-- binding parameter [2] as [VARCHAR]   - [John Doe]
-- binding parameter [3] as [TIMESTAMP] - [Tue Apr 18 17:21:46 EEST 2017]
-- binding parameter [4] as [BIGINT]    - [1]
```

在更新Bid实体时，Hibernate将使用当前JVM时间戳值修改`update_on`列：

###### 例75.`@UpdateTimestamp`更新示例

```java
Bid bid = entityManager.find( Bid.class, 1L );

bid.setUpdatedBy( "John Doe Jr." );
bid.setCents( 160 * 100L );
entityManager.persist( bid );
```

```java
UPDATE Bid SET
    cents = ?,
    updated_by = ?,
    updated_on = ?
where
    id = ?

-- binding parameter [1] as [BIGINT]    - [16000]
-- binding parameter [2] as [VARCHAR]   - [John Doe Jr.]
-- binding parameter [3] as [TIMESTAMP] - [Tue Apr 18 17:49:24 EEST 2017]
-- binding parameter [4] as [BIGINT]    - [1]
```

##### `@ValueGenerationType`元注解 {#mapping-generated-ValueGenerationType}

Hibernate 4.3引入了`@ValueGenerationType`元注解，这是一种声明生成的属性或自定义生成器的新方法。

`@Generated`已经被翻新为使用`@ValueGenerationType`元注解。但是`@ValueGenerationType`公开了比`@Generated`当前支持的特性更多的特性，并且为了利用其中的一些特性，您只需要连接一个新的生成器注释。

如下面的示例所示，在声明用于标记需要特定生成策略的实体属性的自定义注释时，将使用`@ValueGenerationType`元注释。必须将实际的生成逻辑添加到实现`AnnotationValueGeneration`接口的类中。

##### 数据库生成值

例如，假设我们希望通过调用标准ANSI SQL函数`current_timestamp`（而不是触发器或DEFAULT值）来生成时间戳：

###### 例76.用于数据库生成的`ValueGenerationType`映射

```java
@Entity(name = "Event")
public static class Event {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "`timestamp`")
    @FunctionCreationTimestamp
    private Date timestamp;

    //Constructors, getters, and setters are omitted for brevity
}

@ValueGenerationType(generatedBy = FunctionCreationValueGeneration.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface FunctionCreationTimestamp {}

public static class FunctionCreationValueGeneration
        implements AnnotationValueGeneration<FunctionCreationTimestamp> {

    @Override
    public void initialize(FunctionCreationTimestamp annotation, Class<?> propertyType) {
    }

    /**
     * Generate value on INSERT
     * @return when to generate the value
     */
    public GenerationTiming getGenerationTiming() {
        return GenerationTiming.INSERT;
    }

    /**
     * Returns null because the value is generated by the database.
     * @return null
     */
    public ValueGenerator<?> getValueGenerator() {
        return null;
    }

    /**
     * Returns true because the value is generated by the database.
     * @return true
     */
    public boolean referenceColumnInSql() {
        return true;
    }

    /**
     * Returns the database-generated value
     * @return database-generated value
     */
    public String getDatabaseGeneratedReferencedColumnValue() {
        return "current_timestamp";
    }
}
```

当持久化Event实体时，Hibernate生成以下SQL语句：

```java
INSERT INTO Event ("timestamp", id)
VALUES (current_timestamp, 1)
```

你可以看到，这`current_timestamp`值用于指定`timestamp`列的值。

##### 内存中生成的值

如果需要在内存中生成时间戳值，则必须使用以下映射：

###### 例77.用于内存值生成的`ValueGenerationType`映射

```java
@Entity(name = "Event")
public static class Event {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "`timestamp`")
    @FunctionCreationTimestamp
    private Date timestamp;

    //Constructors, getters, and setters are omitted for brevity
}

@ValueGenerationType(generatedBy = FunctionCreationValueGeneration.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface FunctionCreationTimestamp {}

public static class FunctionCreationValueGeneration
        implements AnnotationValueGeneration<FunctionCreationTimestamp> {

    @Override
    public void initialize(FunctionCreationTimestamp annotation, Class<?> propertyType) {
    }

    /**
     * Generate value on INSERT
     * @return when to generate the value
     */
    public GenerationTiming getGenerationTiming() {
        return GenerationTiming.INSERT;
    }

    /**
     * Returns the in-memory generated value
     * @return {@code true}
     */
    public ValueGenerator<?> getValueGenerator() {
        return (session, owner) -> new Date( );
    }

    /**
     * Returns false because the value is generated by the database.
     * @return false
     */
    public boolean referenceColumnInSql() {
        return false;
    }

    /**
     * Returns null because the value is generated in-memory.
     * @return null
     */
    public String getDatabaseGeneratedReferencedColumnValue() {
        return null;
    }
}
```

当持久化Event实体时，Hibernate生成以下SQL语句：

```java
INSERT INTO Event ("timestamp", id)
VALUES ('Tue Mar 01 10:58:18 EET 2016', 1)
```

可以看到，`new Date()`对象值用于指定`timestamp`列值。

