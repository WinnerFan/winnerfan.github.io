---
layout: post
title: Spring Boot Usage
tags: Spring
---
## Spring Boot Starter
@SpringBootApplication等价于@Configuration基于Java配置，@ComponentScan组件扫描，@EnableAutoConfiguration自动配置

## XML
### profile
application.properties中`spring.profiles.active=prd`，类上添加`@profile("prd")`.使用application-prd.properties中，否则默认使用application.properties中

## Component
### entity
```
@Entity //定义实体类
public class Book {
    @Id //唯一标识
    @GeneratedValue(strategy=GenerateionType.AUTO) //自动生成
    private Long id;
}
```
### properties
新建类对应一类properties，使用时自动注入
```
@Component
@ConfigurationProperties("amazon") //批量注入，前缀为amazon的属性
public class AmazonProperties {
    private String associateId; // 对应amazon.associateId或amazon.associate_id或amazon.associate-id
    //getter setter
}
```
## Repository
### 继承JpaRepository
JPQL不支持INSERT操作，除非原生SQL
```
public interface ReadingListRepository extends JpaRepository<Book, Long>{
    // findByXX自动生成语句，例如findByXXLike, findByXXNotLike, findByXXNot, findByXXIn, findByXXOrderByYYDesc, findByXXAndYY等
    // 相当于select * from books where reader=?
    List<Book> findByReader(String reader);
    
    @Modifying //通知jpa是一个update或delete
    @Transaction //事务
    @Query("update Book b set b.isbn=?2 where b.reader=?1")
    int updateReader1(String reader, String isbn);
    
    @Modifying //通知jpa是一个update或delete
    @Transaction //事务
    @Query("update Book b set b.isbn=:isbn where b.reader=:reader")
    int updateReader2(@Param("reader") String reader, @Param("isbn") String isbn);
    
    @Modifying //通知jpa是一个update或delete，必须
    @Transaction //事务
    @Query("update Book b set b.isbn=?2 where b.reader like ?1%")
    int updateReader3(String reader, String isbn);
    
    @Query(value = "insert into reader ... ?1", nativeQuery = true)
    int insertReader(String reader, String isbn);
}
```
### JdbcTemplate
```
public class BookRowMapper implements RowMapper {
    @Override
    public Object mapRow(ResultSet set, int index) throws SQLException {
        Book book = new Book();
        book.setId(set.getInt("id"));
    }
}
@Repository
public class JdbcReadingListRepository implements ReadingListRepository {
    @Autowired
    JdbcTemplate jdbc;
    public void save(Book book) {
        jdbc.update("insert ... values (?, ?, ?)", book.reader, book.isbn, book.title);
        // jdbc.update("insert ... values (?, ?, ?)", new Object[]{book.reader, ...}, new int[]{java.sql.Types.VARCHAR, ...})
    }
    public List<Book> getBooks {
        return jdbc.query("select * ...", new BookRowMapper());
    }
}
```
### MyBatis

## Controller
### GET
```
@RequestMapping(value="/{reader}", method=RequestMethod.GET)
public String readersBooks(@PathVariable("reader") String reader) {
    ...
}
```

## Test
### 集成测试
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes=AddressBookConfiguration.class)
public class AddressServiceTests {
    @Autowired
    ...
    @Test
    ...
}
```
### 测试Web
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes=AddressBookConfiguration.class)
@WebAppConifguration
public class MockMvcWebTests {
    @Autowired
    private WebApplicationContext webContext;
    private MockMvc mockMvc;
    @Before
    public void setupMockMvc() {
        mockMvc = MockMvcBuilders.webAppContextSetup(webContext).build();
    }
    @Test
    public void homePage() throw Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/readingList")
            .andExpect(MockMvcResultMatchers.status().isOk()));
    }
}
```
### 测试运行中的应用程序
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes=AddressBookConfiguration.class)
@WebIntegrationTest
```