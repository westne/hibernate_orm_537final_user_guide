Hibernate支持各种Java Date/Time类作为持久域模型实体属性被映射。SQL标准定义了三种Date/Time类型：

`DATE`

代表一个日历日期，存储年月日。 JDBC等价是`java.sql.Date`

`TIME`

代表一天中的时间，存储时分秒。JDBC等价是`java.sql.Time`

`TIMESTAMP`

存储一个DATE和一个TIME加纳秒。JDBC等价是`java.sql.Timestamp`

> 为了避免对`java.sql`包的依赖，通常使用`java.util`或`java.time`包的Date/Time类。

当`java.sql`类定义与SQL Date/Time数据类型的直接关联时，`java.util`或`java.time`属性需要显式地将SQL类型关联标记为`@Temporal` 注解。通过这种方式，可以将`java.util.Date`或`java.util.Calendar`映射到SQL `DATE`、`TIME`或`TIMESTAMP`类型。





