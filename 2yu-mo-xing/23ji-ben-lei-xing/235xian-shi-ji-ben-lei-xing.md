有时您希望以不同的方式处理特定的属性。偶尔Hibernate会隐式地选择您不想要的`BasicType`（由于某些原因，您不想调整`BasicTypeRegistry`）。

在这些情况下，必须通过`org.hibernate.annotations.Type`注释显式地告诉Hibernate要使用的`BasicType`。

