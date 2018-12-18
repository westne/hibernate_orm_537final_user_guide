我们之前说过，Hibernate类型不是Java类型，也不是SQL类型，但它理解这两种类型并执行它们之间的编组。但是看看前面示例中的基本类型映射，Hibernate如何知道使用`org.hibernate.type.StringType`映射`java.lang.String`属性，或者使用`org.hibernate.type.IntegerType`映射`java.lang.Integer`属性？

