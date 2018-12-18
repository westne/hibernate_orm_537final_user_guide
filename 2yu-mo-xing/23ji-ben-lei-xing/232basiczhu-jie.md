Strictly speaking, a basic type is denoted by the`javax.persistence.Basic`annotation. Generally speaking, the`@Basic`annotation can be ignored, as it is assumed by default. Both of the following examples are ultimately the same.

###### 示例3.`@Basic`显式声明

```java
@Entity(name = "Product")
public class Product {

	@Id
	@Basic
	private Integer id;

	@Basic
	private String sku;

	@Basic
	private String name;

	@Basic
	private String description;
}
```



