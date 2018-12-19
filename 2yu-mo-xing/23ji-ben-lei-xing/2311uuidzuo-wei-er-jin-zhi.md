如前所述，UUID属性的默认映射。使用`java.util.UUID#getMostSignificantBits`和`java.util.UUID#getLeastSignificantBits`将UUID映射到byte\[\]，并将其存储为`BINARY`数据。

选择默认值只是因为从存储角度来看，它通常更有效。

