基本值类型通常将单个数据库列映射为单个、非聚合的Java类型。Hibernate提供了许多内置的基本类型，它们遵循JDBC规范推荐的自然映射。

内部Hibernate在需要解析特定的`org.hibernate.type.Type`时使用基本类型的注册表。

