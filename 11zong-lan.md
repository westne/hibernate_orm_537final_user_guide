![](/assets/data_access_layers.svg)

作为一个ORM解决方案，Hibernate有效地“介于”Java应用程序数据访问层和关系数据库之间，如上面的图表所示。Java应用程序利用Hibernate API来加载、存储、查询其域数据。这里我们将介绍基本的HibernateAPI。这将是一个简短的介绍；稍后我们将详细讨论这些约定。

作为一个JPA提供者，Hibernate实现了Java持久化API规范，JPA接口和Hibernate具体实现之间的关联可以在下面的图表中可视化：

###### ![](/assets/JPA_Hibernate.svg)SessionFactory \(`org.hibernate.SessionFactory`\)

应用程序域模型到数据库的映射的一个线程安全（和不可变）的表示。充当`org.hibernate.Session`实例的工厂类。`EntityManagerFactory`是JPA层面等效于`SessionFactory`的，基本上，这两者会聚到相同的SessionFactory实现中。

创建`SessionFactory`非常昂贵，因此，对于任何给定的数据库，应用程序都应该只有一个相关联的`SessionFactory`。`SessionFactory`维护Hibernate在所有`Session(s)`中使用的服务，如二级缓存、连接池、事务系统集成等。

###### Session \(`org.hibernate.Session`\)

在概念上对“工作单元”[PoEAA](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#PoEAA)进行建模的单线程、短期对象。在JPA命名法中，`Session`由`EntityManager`表示。

在幕后，Hibernate `Session`包装JDBC `java.sql.Connection`，并作为`org.hibernate.Transaction`实例的工厂。它维护应用程序域模型的一般“可重复读”持久性上下文（第一级缓存）。

###### Transaction \(`org.hibernate.Transaction`\)

应用程序用来划分单个物理事务边界的单线程、短期对象。`EntityTransaction`是JPA等价物，二者都充当抽象API，用于将应用程序与正在使用的底层事务系统（JDBC或JTA）隔离。



