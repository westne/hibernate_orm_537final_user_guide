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



