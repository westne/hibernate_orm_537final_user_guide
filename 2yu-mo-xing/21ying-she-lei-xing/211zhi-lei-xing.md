值类型是不定义其自身生命周期的一段数据。实际上，它是由定义其生命周期的实体所拥有的。

从另一个角度看，实体的所有状态都完全由值类型组成。这些状态字段或JavaBean属性称为_持久属性_。`Contact`类的持久属性是值类型。

值类型进一步分为三个子类别：

###### 基本类型

在映射Contact表时，除了name之外的所有属性都是基本类型。基本类型将在[基本类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#basic)中详细讨论。

###### 可嵌入类型

name属性是可嵌入类型的示例，在[可嵌入类型](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#embeddables)中将详细讨论该属性。

###### 集合类型

尽管在上述示例中没有特性，但是在值类型中，集合类型也是不同的类别。集合类型将在[集合](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#collections)中进一步讨论。

