就像`@Where`注释一样，`@WhereJoinTable`用于使用联合表（例如`@ManyToManyAssociation`）筛选集合。

###### 例87.`@WhereJoinTable`映射示例

```java
@Entity(name = "Book")
public static class Book {

	@Id
	private Long id;

	private String title;

	private String author;

	@ManyToMany
	@JoinTable(
		name = "Book_Reader",
		joinColumns = @JoinColumn(name = "book_id"),
		inverseJoinColumns = @JoinColumn(name = "reader_id")
	)
	@WhereJoinTable( clause = "created_on > DATEADD( 'DAY', -7, CURRENT_TIMESTAMP() )")
	private List<Reader> currentWeekReaders = new ArrayList<>( );

	//Getters and setters omitted for brevity

}

@Entity(name = "Reader")
public static class Reader {

	@Id
	private Long id;

	private String name;

	//Getters and setters omitted for brevity

}
```

```java
create table Book (
    id bigint not null,
    author varchar(255),
    title varchar(255),
    primary key (id)
)

create table Book_Reader (
    book_id bigint not null,
    reader_id bigint not null
)

create table Reader (
    id bigint not null,
    name varchar(255),
    primary key (id)
)

alter table Book_Reader
    add constraint FKsscixgaa5f8lphs9bjdtpf9g
    foreign key (reader_id)
    references Reader

alter table Book_Reader
    add constraint FKoyrwu9tnwlukd1616qhck21ra
    foreign key (book_id)
    references Book

alter table Book_Reader
    add created_on timestamp
    default current_timestamp
```



