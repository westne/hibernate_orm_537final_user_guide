我们之前说过，Hibernate类型不是Java类型，也不是SQL类型，但它理解这两种类型并执行它们之间的编组。但是看看前面示例中的基本类型映射，Hibernate如何知道使用`org.hibernate.type.StringType`映射`java.lang.String`属性，或者使用`org.hibernate.type.IntegerType`映射`java.lang.Integer`属性？

答案在于Hibernate内部的一个名为`org.hibernate.type.BasicTypeRegistry`的服务，它基本上维护一个由名称键控的`org.hibernate.type.BasicType`（`org.hibernate.type.Type`专门化）实例的映射。

稍后我们将在[显式基本类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-type-annotation)部分中看到，我们可以显式地告诉Hibernate为特定属性使用哪个BasicType。但是，首先，让我们探讨隐式解析如何工作，以及应用程序如何调整隐式解析。

> 深入讨论`BasicTypeRegistry`以及向其贡献类型的所有不同方式超出了本文档的范围。详细信息请参阅集成指南。

举个例子，我们之前在Product\#sku中看到的String属性。由于没有显式的类型映射，Hibernate查找`BasicTypeRegistry`以查找`java.lang.String`的注册映射。这回到本章开始时在表中看到的“BasicTypeRegistry键”列。

作为`BasicTypeRegistry`中的基线，Hibernate遵循JDBC对Java类型的推荐映射。JDBC建议将String映射到VARCHAR，这是StringType处理的精确映射。所以这就是字符串`BasicTypeRegistry`中的基线映射。

应用程序还可以在启动期间使用`MetadataBuilder#applyBasicType`方法，或者继承（添加新的`BasicType`注册）或重写（替换现有的`BasicType`注册）`MetadataBuilder#applyTypes`方法之一。有关详细信息，请参阅[自定义基本类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic-custom-type)部分。

