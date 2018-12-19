> 当使用一个PostgreSQL的方言，这成为默认的UUID映射。

使用PostgreSQL的特定UUID数据类型映射UUID。PostgreSQL JDBC驱动程序选择将其UUID类型映射到`OTHER`代码。注意，这可能会造成困难，因为驱动程序选择将许多不同的数据类型映射到`OTHER`数据类型。



