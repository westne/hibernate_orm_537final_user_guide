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

在上面的示例中，当前周`Reader`实体包含在`currentWeekReaders`集合中，该集合使用`@WhereJoinTable`注解根据提供的SQL子句筛选合并的表行。

考虑到以下两个Book\_Reader条目被添加到我们的系统中：

###### 例88.`@WhereJoinTable`测试数据

```java
Book book = new Book();
book.setId( 1L );
book.setTitle( "High-Performance Java Persistence" );
book.setAuthor( "Vad Mihalcea" );
entityManager.persist( book );

Reader reader1 = new Reader();
reader1.setId( 1L );
reader1.setName( "John Doe" );
entityManager.persist( reader1 );

Reader reader2 = new Reader();
reader2.setId( 2L );
reader2.setName( "John Doe Jr." );
entityManager.persist( reader2 );

statement.executeUpdate(
	"INSERT INTO Book_Reader " +
	"	(book_id, reader_id) " +
	"VALUES " +
	"	(1, 1) "
);
statement.executeUpdate(
	"INSERT INTO Book_Reader " +
	"	(book_id, reader_id, created_on) " +
	"VALUES " +
	"	(1, 2, DATEADD( 'DAY', -10, CURRENT_TIMESTAMP() )) "
);
```



