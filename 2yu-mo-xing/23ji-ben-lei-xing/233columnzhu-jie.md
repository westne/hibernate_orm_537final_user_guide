JPA定义了隐式确定表和列的名称的规则。有关隐式命名的详细讨论，请参阅[命名](http://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#naming)。

对于基本类型属性，隐式命名规则是列名与属性名相同。如果该隐式命名规则不满足您的要求，则可以显式地告诉Hibernate（和其他提供者）要使用的列名。

###### 例5.显式列命名

```java
@Entity(name = "Product")
public class Product {

	@Id
	private Integer id;

	private String sku;

	private String name;

	@Column( name = "NOTES" )
	private String description;
}
```



