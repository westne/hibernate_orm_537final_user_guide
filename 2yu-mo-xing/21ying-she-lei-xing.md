Hibernate同时理解应用程序数据的Java和JDBC表示。从数据库读取/写入这些数据的能力是Hibernate_类型_的功能。在这种用法中，类型是`org.hibernate.type.Type`接口的实现。Hibernate类型还描述了Java类型的各种行为方面，例如如何检查相等性、如何克隆值等。

> _类型的用法_
>
> Hibernate类型既不是Java类型，也不是SQL数据类型。它提供了关于这两个方面的信息以及理解它们之间的编组。
>
> 当遇到Hibernate讨论中的术语类型时，它可能指代Java类型、JDBC类型或Hibernate类型，根据上下文。

为了帮助理解类型分类，让我们看一个我们希望映射的简单表和域模型。

