###### 表1.标准基本类型

| Hibernate type \(org.hibernate.type package\) | JDBC type | Java type | BasicTypeRegistry key\(s\) |
| :--- | :--- | :--- | :--- |
| StringType | VARCHAR | java.lang.String | string, java.lang.String |
| MaterializedClob | CLOB | java.lang.String | materialized\_clob |
| TextType | LONGVARCHAR | java.lang.String | text |
| CharacterType | CHAR | char, java.lang.Character | char, java.lang.Character |
| BooleanType | BIT | boolean, java.lang.Boolean | boolean, java.lang.Boolean |
| NumericBooleanType | INTEGER, 0 is false, 1 is true | boolean, java.lang.Boolean | numeric\_boolean |
| YesNoType | CHAR, 'N'/'n' is false, 'Y'/'y' is true. The uppercase value is written to the database. | boolean, java.lang.Boolean | yes\_no |
| TrueFalseType | CHAR, 'F'/'f' is false, 'T'/'t' is true. The uppercase value is written to the database. | boolean, java.lang.Boolean | true\_false |
| ByteType | TINYINT | byte, java.lang.Byte | byte, java.lang.Byte |
| ShortType | SMALLINT | short, java.lang.Short | short, java.lang.Short |
| IntegerTypes | INTEGER | int, java.lang.Integer | int, java.lang.Integer |
| LongType | BIGINT | long, java.lang.Long | long, java.lang.Long |
| FloatType | FLOAT | float, java.lang.Float | float, java.lang.Float |
| DoubleType | DOUBLE | double, java.lang.Double | double, java.lang.Double |
| BigIntegerType | NUMERIC | java.math.BigInteger | big\_integer, java.math.BigInteger |
| BigDecimalType | NUMERIC | java.math.BigDecimal | big\_decimal, java.math.bigDecimal |
| TimestampType | TIMESTAMP | java.sql.Timestamp | timestamp, java.sql.Timestamp |
| TimeType | TIME | java.sql.Time | time, java.sql.Time |
| DateType | DATE | java.sql.Date | date, java.sql.Date |
| CalendarType | TIMESTAMP | java.util.Calendar | calendar, java.util.Calendar |
| CalendarDateType | DATE | java.util.Calendar | calendar\_date |
| CalendarTimeType | TIME | java.util.Calendar | calendar\_time |
| CurrencyType | VARCHAR | java.util.Currency | currency, java.util.Currency |
| LocaleType | VARCHAR | java.util.Locale | locale, java.utility.locale |
| TimeZoneType | VARCHAR, using the TimeZone ID | java.util.TimeZone | timezone, java.util.TimeZone |
| UrlType | VARCHAR | java.net.URL | url, java.net.URL |
| ClassType | VARCHAR \(class FQN\) | java.lang.Class | class, java.lang.Class |
| BlobType | BLOB | java.sql.Blob | blob, java.sql.Blob |
| ClobType | CLOB | java.sql.Clob | clob, java.sql.Clob |
| BinaryType | VARBINARY | byte\[\] | binary, byte\[\] |
| MaterializedBlobType | BLOB | byte\[\] | materialized\_blob |
| ImageType | LONGVARBINARY | byte\[\] | image |
| WrapperBinaryType | VARBINARY | java.lang.Byte\[\] | wrapper-binary, Byte\[\], java.lang.Byte\[\] |
| CharArrayType | VARCHAR | char\[\] | characters, char\[\] |
| CharacterArrayType | VARCHAR | java.lang.Character\[\] | wrapper-characters, Character\[\], java.lang.Character\[\] |
| UUIDBinaryType | BINARY | java.util.UUID | uuid-binary, java.util.UUID |
| UUIDCharType | CHAR, can also read VARCHAR | java.util.UUID | uuid-char |
| PostgresUUIDType | PostgreSQL UUID, through Types\#OTHER, which complies to the PostgreSQL JDBC driver definition | java.util.UUID | pg-uuid |
| SerializableType | VARBINARY | implementors of java.lang.Serializable | Unlike the other value types, multiple instances of this type are registered. It is registered once under java.io.Serializable, and registered under the specific java.io.Serializable implementation class names. |
| StringNVarcharType | NVARCHAR | java.lang.String | nstring |
| NTextType | LONGNVARCHAR | java.lang.String | ntext |
| NClobType | NCLOB | java.sql.NClob | nclob, java.sql.NClob |
| MaterializedNClobType | NCLOB | java.lang.String | materialized\_nclob |
| PrimitiveCharacterArrayNClobType | NCHAR | char\[\] | N/A |
| CharacterNCharType | NCHAR | java.lang.Character | ncharacter |
| CharacterArrayNClobType | NCLOB | java.lang.Character\[\] | N/A |

###### 表2.Java 8基本类型

| Hibernate type \(org.hibernate.type package\) | JDBC type | Java type | BasicTypeRegistry key\(s\) |
| :--- | :--- | :--- | :--- |
| DurationType | BIGINT | java.time.Duration | Duration, java.time.Duration |
| InstantType | TIMESTAMP | java.time.Instant | Instant, java.time.Instant |
| LocalDateTimeType | TIMESTAMP | java.time.LocalDateTime | LocalDateTime, java.time.LocalDateTime |
| LocalDateType | DATE | java.time.LocalDate | LocalDate, java.time.LocalDate |
| LocalTimeType | TIME | java.time.LocalTime | LocalTime, java.time.LocalTime |
| OffsetDateTimeType | TIMESTAMP | java.time.OffsetDateTime | OffsetDateTime, java.time.OffsetDateTime |
| OffsetTimeType | TIME | java.time.OffsetTime | OffsetTime, java.time.OffsetTime |
| ZonedDateTimeType | TIMESTAMP | java.time.ZonedDateTime | ZonedDateTime, java.time.ZonedDateTime |

###### 表3.Hibernate空间基本类型

| Hibernate type \(org.hibernate.spatial package\) | JDBC type | Java type | BasicTypeRegistry key\(s\) |
| :--- | :--- | :--- | :--- |
| JTSGeometryType | depends on the dialect | com.vividsolutions.jts.geom.Geometry | jts\_geometry, or the class name of Geometry or any of its subclasses |
| GeolatteGeometryType | depends on the dialect | org.geolatte.geom.Geometry | geolatte\_geometry, or the class name of Geometry or any of its subclasses |

> 要使用这些hibernate-space类型，必须将`hibernate-spatial`依赖项添加到类路径中，并使用`org.hibernate.spatial.SpatialDialect`实现。有关空间类型的详细信息，请参阅[空间](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#spatial)。

这些映射由Hibernate内的一个名为`org.hibernate.type.BasicTypeRegistry`的服务管理，该服务基本上维护`org.hibernate.type.BasicType`\(`org.hibernate.type.Type`专门化\)实例的映射。这就是前面表中的“BasicTypeRegistry键”列的目的。



