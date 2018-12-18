Hibernate使得开发人员创建自己的基本类型映射类型相对容易。例如，您可能希望将`java.util.BigInteger`类型的属性持久化到`VARCHAR`列，或者支持全新的类型。

开发自定义类型有两种方法：

* 实现`BasicType`并注册它

* 实现不需要类型注册的`UserType`

作为说明不同方法的一种手段，让我们考虑一个用例，其中需要支持`java.util.BitSet`映射作为`VARCHAR`存储。

