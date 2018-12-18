当一个实体没有显式地命名它映射到的数据库表时，我们需要隐式地确定该表名称。或者当一个特定的属性没有显式地命名它映射到的数据库列时，我们需要隐式地确定该列名称。这里有org.hibernate.boot.model.naming.ImplicitNamingStrategy约定的作用的示例，用于在映射没有提供显式名称时确定逻辑名称。![](/assets/implicit_naming_strategy_diagram.svg)Hibernate直接定义了多个ImplicitNamingStrategy实现。应用程序还可以自由地使用插件自定义实现。

有多种方法可以指定要使用的ImplicitNamingStrategy。首先，应用程序可以使用hibernate.implicit\_naming\_strategy配置设置指定实现，该设置接受：

* 为开箱即用的实现预先定义的“缩写”

        `default`

        对于`org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl`- 一个`jpa`的别称

        `jpa`

        对于`org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl`- JPA 2.0兼容命名策略

        `legacy-hbm`

        对于`org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyHbmImpl`- 兼容hibernate原生命名策略

        `legacy-jpa`







* 引用实现org.hibernate.boot.model.naming.ImplicitNamingStrategy约定的类
* 实现org.hibernate.boot.model.naming.ImplicitNamingStrategy约定的类的FQN

其次，应用程序和集成可以利用org.hibernate.boot.MetadataBuilder\#applyImplicitNamingStrategy指定要使用的ImplicitNamingStrategy。有关引导的其他详细信息，请参阅引导程序。


