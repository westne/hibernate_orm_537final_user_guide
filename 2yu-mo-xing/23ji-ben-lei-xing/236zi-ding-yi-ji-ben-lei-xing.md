Hibernate使得开发人员创建自己的基本类型映射类型相对容易。例如，您可能希望将`java.util.BigInteger`类型的属性持久化到`VARCHAR`列，或者支持全新的类型。

开发自定义类型有两种方法：

* 实现`BasicType`并注册它

* 实现不需要类型注册的`UserType`

作为说明不同方法的一种手段，让我们考虑一个用例，其中需要支持`java.util.BitSet`映射作为`VARCHAR`存储。

###### 实现BasicType

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



