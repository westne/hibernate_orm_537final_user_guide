术语[域模型](https://en.wikipedia.org/wiki/Domain_model)来源于数据建模领域。它是最终描述您所从事的[问题领域](https://en.wikipedia.org/wiki/Problem_domain)的模型。有时您还会听到_持久类_这个术语。

最终，应用程序域模型是ORM的中心特征。它们构成了您希望映射的类。如果这些类遵循普通的Java对象（POJO）/JavaBean编程模型，Hibernate效果最好。然而，这些规则都不是严格的要求。实际上，Hibernate对持久对象的性质几乎不作任何假设。可以用其他方式表示域模型（例如，使用`java.util.Map`实例的树）。

历史上使用Hibernate的应用程序会为此使用其专有的XML映射文件格式。随着JPA的到来，现在大多数信息都以一种使用注解（和/或标准化XML格式）跨ORM/JPA提供程序可移植的方式定义。本章将尽可能集中于JPA映射。对于JPA不支持的Hibernate映射特性，我们更喜欢Hibernate扩展注解。

