Hibernate支持Java枚举类的映射作为基本值类型，以许多不同的方式。

#### `@Enumerated`

最初的与JPA兼容的映射枚举的方法是通过`@Enumerate`或`@MapKeyEnumerated`作为映射键注释，其原理是枚举值根据javax.persistence.EnumType所指示的两种策略之一进行存储：

    `ORDINAL`

    根据枚举类中枚举值的顺序位置存储，如`Java.Lang.Enum#ordinal`所指示的。

    `STRING`

    按照枚举值的名称存储，如`java.lang.Enum#name`所指示的。

