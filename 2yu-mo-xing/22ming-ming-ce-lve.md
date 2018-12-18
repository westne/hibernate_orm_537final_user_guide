对象模型到关系数据库的部分映射是将名称从对象模型映射到相应的数据库名称。Hibernate将这看作两个阶段的过程：

* 第一阶段是从域模型映射中确定适当的逻辑名称。逻辑名称可以由用户显式指定（例如`@Column`或`@Table`），或者可以由Hibernate通过[ImplicitNamingStrategy](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#ImplicitNamingStrategy)约定隐式确定。
* 其次是将此逻辑名称解析为物理名称，物理名称由[PhysicalNamingStrategy ](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#PhysicalNamingStrategy)约定定义。

> ###### 历史NamingStrategy约定
>
> 在历史上，Hibernate只定义了一个`org.hibernate.cfg.NamingStrategy`.这个单独的NamingStrategy约定实际上结合了现在单独建模为ImplicitNamingStrategy和PhysicalNamingStrategy的分别的关注点。
>
> 此外，NamingStrategy约定通常不够灵活，不能适当地应用给定的命名“规则”，或者因为API缺乏要决定的信息，或者因为随着API的增长，API确实没有定义好。
>
> 由于这些限制，`org.hibernate.cfg.NamingStrategy`已被弃用，然后被移除，支持ImplicitNamingStrategy和PhysicalNamingStrategy。

核心上，每个命名策略背后的思想是最小化开发人员必须为映射域模型提供的重复信息量。

> ###### JPA兼容性
>
> JPA定义了关于隐式逻辑名称确定的内在规则。如果JPA提供程序的可移植性是一个主要问题，或者您真的很喜欢JPA定义的隐式命名规则，那么一定要坚持使用ImplicitNamingStrategyJpaCompliantImpl（默认）
>
> 此外，JPA没有定义逻辑名称和物理名称之间的分离。遵循JPA规范，逻辑名称**就是**物理名称。如果JPA提供方的可移植性很重要，应用程序应该不选择指定PhysicalNamingStrategy。





