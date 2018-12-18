Hibernate使得开发人员创建自己的基本类型映射类型相对容易。例如，您可能希望将`java.util.BigInteger`类型的属性持久化到`VARCHAR`列，或者支持全新的类型。

开发自定义类型有两种方法：

* 实现`BasicType`并注册它

* 实现不需要类型注册的`UserType`

作为说明不同方法的一种手段，让我们考虑一个用例，其中需要支持`java.util.BitSet`映射作为`VARCHAR`存储。

#### 实现BasicType

第一种方法是直接实现BasicType接口。

> 因为`BasicType`接口有很多方法可以实现，所以如果值存储在单个数据库列中，则继承`AbstractStandardBasicType`或`AbstractSingleColumnStandardBasicType`更加方便。

首先，我们需要继承`AbstractSingleColumnStandardBasicType`像这样：

###### 示例7.自定义`BasicType`实现

```java
public class BitSetType
        extends AbstractSingleColumnStandardBasicType<BitSet>
        implements DiscriminatorType<BitSet> {

    public static final BitSetType INSTANCE = new BitSetType();

    public BitSetType() {
        super( VarcharTypeDescriptor.INSTANCE, BitSetTypeDescriptor.INSTANCE );
    }

    @Override
    public BitSet stringToObject(String xml) throws Exception {
        return fromString( xml );
    }

    @Override
    public String objectToSQLString(BitSet value, Dialect dialect) throws Exception {
        return toString( value );
    }

    @Override
    public String getName() {
        return "bitset";
    }

}
```

`AbstractSingleColumnStandardBasicType`需要一个`sqlTypeDescriptor`和一个`javaTypeDescriptor`。`sqlTypeDescriptor`是`VarcharTypeDescriptor.INSTANCE`，因为数据库列是`VARCHAR`。在Java方面，我们需要使用一个`BitSetTypeDescriptor`实例，它可以像这样实现：

###### 例8.自定义`AbstractTypeDescriptor`实现

```java
public class BitSetTypeDescriptor extends AbstractTypeDescriptor<BitSet> {

    private static final String DELIMITER = ",";

    public static final BitSetTypeDescriptor INSTANCE = new BitSetTypeDescriptor();

    public BitSetTypeDescriptor() {
        super( BitSet.class );
    }

    @Override
    public String toString(BitSet value) {
        StringBuilder builder = new StringBuilder();
        for ( long token : value.toLongArray() ) {
            if ( builder.length() > 0 ) {
                builder.append( DELIMITER );
            }
            builder.append( Long.toString( token, 2 ) );
        }
        return builder.toString();
    }

    @Override
    public BitSet fromString(String string) {
        if ( string == null || string.isEmpty() ) {
            return null;
        }
        String[] tokens = string.split( DELIMITER );
        long[] values = new long[tokens.length];

        for ( int i = 0; i < tokens.length; i++ ) {
            values[i] = Long.valueOf( tokens[i], 2 );
        }
        return BitSet.valueOf( values );
    }

    @SuppressWarnings({"unchecked"})
    public <X> X unwrap(BitSet value, Class<X> type, WrapperOptions options) {
        if ( value == null ) {
            return null;
        }
        if ( BitSet.class.isAssignableFrom( type ) ) {
            return (X) value;
        }
        if ( String.class.isAssignableFrom( type ) ) {
            return (X) toString( value);
        }
        throw unknownUnwrap( type );
    }

    public <X> BitSet wrap(X value, WrapperOptions options) {
        if ( value == null ) {
            return null;
        }
        if ( String.class.isInstance( value ) ) {
            return fromString( (String) value );
        }
        if ( BitSet.class.isInstance( value ) ) {
            return (BitSet) value;
        }
        throw unknownWrap( value.getClass() );
    }
}
```

`unwrap`方法用于将`BitSet`作为`PreparedStatement`绑定参数传递的时候，而`wrap`方法用于将JDBC列值对象（在本例中为`String`）转换为实际的映射对象类型（在本例中为`BitSet`）。

必须注册`BasicType`，这可以在引导时完成：

###### 示例9.注册自定义`BasicType`实现

```java
configuration.registerTypeContributor( (typeContributions, serviceRegistry) -> {
    typeContributions.contributeType( BitSetType.INSTANCE );
} );
```

或者使用`MetadataBuilder`

```java
ServiceRegistry standardRegistry =
    new StandardServiceRegistryBuilder().build();

MetadataSources sources = new MetadataSources( standardRegistry );

MetadataBuilder metadataBuilder = sources.getMetadataBuilder();

metadataBuilder.applyBasicType( BitSetType.INSTANCE );
```

随着新的`BitSetType`被注册为`bitset`，实体映射看起来像这样：

###### 示例10.自定义`BasicType`映射

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    @Type( type = "bitset" )
    private BitSet bitSet;

    public Integer getId() {
        return id;
    }

    //Getters and setters are omitted for brevity
}
```

另外，你可以使用`@TypeDef`，并且跳过注册阶段：

###### 示例11.使用`@TypeDef`注册一个自定义类型

```java
@Entity(name = "Product")
@TypeDef(
    name = "bitset",
    defaultForType = BitSet.class,
    typeClass = BitSetType.class
)
public static class Product {

    @Id
    private Integer id;

    private BitSet bitSet;

    //Getters and setters are omitted for brevity
}
```

为了校验新的`BasicType`实现，我们可以进行如下测试：

###### 示例12.持久化自定义`BasicType`

```java
BitSet bitSet = BitSet.valueOf( new long[] {1, 2, 3} );

doInHibernate( this::sessionFactory, session -> {
    Product product = new Product( );
    product.setId( 1 );
    product.setBitSet( bitSet );
    session.persist( product );
} );

doInHibernate( this::sessionFactory, session -> {
    Product product = session.get( Product.class, 1 );
    assertEquals(bitSet, product.getBitSet());
} );
```

当执行这个单元测试，Hibernate生成如下的SQL语句：

###### 示例13.持久化自定义`BasicType`

```SQL
DEBUG SQL:92 -
    insert
    into
        Product
        (bitSet, id)
    values
        (?, ?)

TRACE BasicBinder:65 - binding parameter [1] as [VARCHAR] - [{0, 65, 128, 129}]
TRACE BasicBinder:65 - binding parameter [2] as [INTEGER] - [1]

DEBUG SQL:92 -
    select
        bitsettype0_.id as id1_0_0_,
        bitsettype0_.bitSet as bitSet2_0_0_
    from
        Product bitsettype0_
    where
        bitsettype0_.id=?

TRACE BasicBinder:65 - binding parameter [1] as [INTEGER] - [1]
TRACE BasicExtractor:61 - extracted value ([bitSet2_0_0_] : [VARCHAR]) - [{0, 65, 128, 129}]
```

正如你所见，`BitSetType`类负责Java到SQL和SQL到Java的类型转换。

#### 实现一个`UserType`

第二个方法是实现`UserType`接口。

###### 示例14.自定义`UserType`实现

```java
public class BitSetUserType implements UserType {

    public static final BitSetUserType INSTANCE = new BitSetUserType();

    private static final Logger log = Logger.getLogger( BitSetUserType.class );

    @Override
    public int[] sqlTypes() {
        return new int[] {StringType.INSTANCE.sqlType()};
    }

    @Override
    public Class returnedClass() {
        return BitSet.class;
    }

    @Override
    public boolean equals(Object x, Object y)
            throws HibernateException {
        return Objects.equals( x, y );
    }

    @Override
    public int hashCode(Object x)
            throws HibernateException {
        return Objects.hashCode( x );
    }

    @Override
    public Object nullSafeGet(
            ResultSet rs, String[] names, SharedSessionContractImplementor session, Object owner)
            throws HibernateException, SQLException {
        String columnName = names[0];
        String columnValue = (String) rs.getObject( columnName );
        log.debugv("Result set column {0} value is {1}", columnName, columnValue);
        return columnValue == null ? null :
                BitSetTypeDescriptor.INSTANCE.fromString( columnValue );
    }

    @Override
    public void nullSafeSet(
            PreparedStatement st, Object value, int index, SharedSessionContractImplementor session)
            throws HibernateException, SQLException {
        if ( value == null ) {
            log.debugv("Binding null to parameter {0} ",index);
            st.setNull( index, Types.VARCHAR );
        }
        else {
            String stringValue = BitSetTypeDescriptor.INSTANCE.toString( (BitSet) value );
            log.debugv("Binding {0} to parameter {1} ", stringValue, index);
            st.setString( index, stringValue );
        }
    }

    @Override
    public Object deepCopy(Object value)
            throws HibernateException {
        return value == null ? null :
            BitSet.valueOf( BitSet.class.cast( value ).toLongArray() );
    }

    @Override
    public boolean isMutable() {
        return true;
    }

    @Override
    public Serializable disassemble(Object value)
            throws HibernateException {
        return (BitSet) deepCopy( value );
    }

    @Override
    public Object assemble(Serializable cached, Object owner)
            throws HibernateException {
        return deepCopy( cached );
    }

    @Override
    public Object replace(Object original, Object target, Object owner)
            throws HibernateException {
        return deepCopy( original );
    }
}
```

实体映射看起来如下：

###### 示例15.自定义`UserType`映射

```java
@Entity(name = "Product")
public static class Product {

    @Id
    private Integer id;

    @Type( type = "bitset" )
    private BitSet bitSet;

    //Constructors, getters, and setters are omitted for brevity
}
```

在这个例子中，`UserType`注册在`bitset`名下，完成像这样：

###### 示例16.注册一个自定义`UserType`实现

```java
configuration.registerTypeContributor( (typeContributions, serviceRegistry) -> {
    typeContributions.contributeType( BitSetUserType.INSTANCE, "bitset");
} );
```

或者使用`MetadataBuilder`

```java
ServiceRegistry standardRegistry =
    new StandardServiceRegistryBuilder().build();

MetadataSources sources = new MetadataSources( standardRegistry );

MetadataBuilder metadataBuilder = sources.getMetadataBuilder();

metadataBuilder.applyBasicType( BitSetUserType.INSTANCE, "bitset" );
```

> 正如`BasicType`，你也能够使用一个简单名称注册`UserType`。
>
> 不注册，`UserType`映射需要全定义名：
>
> ```java
> @Type( type = "org.hibernate.userguide.mapping.basic.BitSetUserType" )
> ```

当对`BitSetUserType`实体映射运行先前的测试用例时，Hibernate执行以下SQL语句：

示例17. 持久化自定义`BasicType`

```SQL
DEBUG SQL:92 -
    insert
    into
        Product
        (bitSet, id)
    values
        (?, ?)

DEBUG BitSetUserType:71 - Binding 1,10,11 to parameter 1
TRACE BasicBinder:65 - binding parameter [2] as [INTEGER] - [1]

DEBUG SQL:92 -
    select
        bitsetuser0_.id as id1_0_0_,
        bitsetuser0_.bitSet as bitSet2_0_0_
    from
        Product bitsetuser0_
    where
        bitsetuser0_.id=?

TRACE BasicBinder:65 - binding parameter [1] as [INTEGER] - [1]
DEBUG BitSetUserType:56 - Result set column bitSet2_0_0_ value is 1,10,11
```



