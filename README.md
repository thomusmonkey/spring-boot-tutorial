# Spring Boot Application Tutorial

### 1. Overview

ในบทความนี้เราทำจะสร้าง Backend Application ด้วย Java framework อย่าง Spring Boot กัน ซึ่ง Application ที่จะสร้างนี้จะเป็น RESTful API ที่สามารถรองรับ HTTP Method ทั้ง POST, GET, PUT และ DELETE ตามคำร้องของผู้ขอใช้บริการ โดยใช้ CRUD Method เพื่อทำการ Create, Read, Update และ Delete  ข้อมูลในฐานข้อมูลซึ่งจะเป็น MySQL หรือ Mariadb

 
โดยจะออกแบบ API โดยแบ่งออกเป็น 3 Layers คือ

- Presentation Layer — เป็นชั้นของ Rest Controller ที่จะร้องขอ/ส่งต่อข้อมูลไปยัง Service layer จะใช้ @RestController annotation

- Service Layer — เป็นชั้นที่มี Business Logic เพื่อให้ข้อมูลกับ Presentation Layer และร้องขอข้อมูลจาก Persistence layer จะใช้ @Service annotation

- Persistence Layer — เป็นชั้นที่ติดต่อกับ Database โดยตรงและให้ข้อมูลกับ Service layer

### 2. Setup

### JDK

- [Download JDK 17](https://www.oracle.com/java/technologies/downloads/?er=221886#java17) แล้วกด Install แล้วรอสักครู่
- หลังจากติดตั้ง JDK เรียบร้อยให้ set JAVA_HOME ให้เรียบร้อย

```
export JAVA_HOME=$(/usr/libexec/java_home)
export PATH=$PATH:$JAVA_HOME\bin;
```
ลองตรวจสอบ java version ที่ download มา
```
java -version
```

![alt text](image-11.png)

### Spring Initializr

เราจะมาสร้าง Spring Boot Project ซึ่งสามารถทำได้หลายวิธี แต่วิธีที่ง่ายที่สุดคือใช้ [Spring Initializr](https://start.spring.io/) ซึ่งเป็นเว็บที่ใช้สำหรับสร้าง Project เริ่มต้นให้เรา ซึ่งจะมีให้เราเลือกว่าจะใช้ dependency อะไรบ้าง แล้วกด Generate เพียงเท่านี้เราก็ได้ Spring Boot project แล้ว ก็สามารถเลือกได้ว่าจะเป็น Maven หรือ Gradle project ใช้ภาษา Java หรือ Kotlin ก็ได้ แต่ในบทความนี้จะใช้ Maven project และเลือก dependency เป็น Web, JPA, MariaDB และ ​Lombok ดังนี้

![alt text](image-3.png)


### Intellij
เป็น tool สำหรับใช้พัฒนา Spring Boot Project โดยจะต้องเปิด unzip file ที่เพิ่ง download มาและจัดการตามตัวอย่างดังต่อไปนี้ หากยังไม่มีจะต้อง [Download Intellij](https://www.jetbrains.com/idea/download/)

![alt text](image-1.png)
เมื่อทำการเลือก Project from Existing Sources... จะมี pop up ขึ้นมาให้เราเลือกเป็น import project from external model และเลือกเป็น Maven และกด ​Create

![alt text](image-2.png)

### Database
ก่อนอื่นเรามาลง database กันก่อน สำหรับไว้ ​CRUD ข้อมูล ซึ่งในที่นี้เราจะสร้าง database mariadb server ด้วย docker ซึ่งถ้าเราไม่มี docker ให้เราทำการ [Download Docker Desktop](https://www.docker.com/products/docker-desktop/) ขึ้นมาก่อน เมื่อทำการ download และติดตั้งเรียบร้อยแล้ว ให้ทำการ start app ขึ้นมา หลังจาก start เรียบร้อยแล้วให้เราลองสร้าง folder จาก folder Document แล้วลองทำตามขั้นตอนดังนี้

create folder ขึ้นมา
```
mkdir marirdb
cd marirdb
```
ให้เพิ่ม file `docker-compose.yml` และเพิ่ม config ใน file ดังนี้
```yml
version: '2.1'

services:
  mariadb:
    image: docker.io/mariadb:11.1.2
    environment:
      - MARIADB_ROOT_USER=root
      - MARIADB_ROOT_PASSWORD=P@ssw0rd
      - MARIADB_USER=dev
      - MARIADB_PASSWORD=P@ssw0rd#
      - MARIADB_PORT_NUMBER=3306
      - MARIADB_DATABASE=demo
      - TZ=Asia/Bangkok
    volumes:
      # - 'mariadb_data:/bitnami/mariadb'
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - 23306:3306
    healthcheck:
      test: 'exit 0'
```
หลังจากนั้นให้ create folder ขึ้นมาดังนี้
```
mkdir db
cd db
```
และให้เพิ่ม file `init.sql` และเพิ่ม config ดังต่อไปนี้
```sql
SET NAMES utf8mb4;

CREATE DATABASE IF NOT EXISTS `demo` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE `demo`;

# Dump of table book
# ------------------------------------------------------------

DROP TABLE IF EXISTS `book`;

CREATE TABLE `book` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(50) DEFAULT NULL,
  `author` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


# Dump of table order
# ------------------------------------------------------------

DROP TABLE IF EXISTS `order`;

CREATE TABLE `order` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `bookId` int(11) DEFAULT NULL,
  `orderNumber` varchar(20) DEFAULT NULL,
  `description` varchar(50) DEFAULT NULL,
  `createDatetime` datetime DEFAULT NULL,
  `updateDatetime` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```
หลังจากนั้นให้ cd กลับมาที่ folder mariadb และพิมพ์คำสั่งดังต่อไปนี้ เพื่อสร้าง database server ขึ้นมา โดยเราสามารถเชื่อมต่อ โดยใช้ `localhost:23306`
```
docker-compose up -d
```

### 3. Application Configuration

เพิ่ม config ดังต่อไปนี้ใน file `application.properties`

```.properties
spring.application.name=demo

server.port=8080

spring.datasource.jdbc-url=jdbc:mariadb://localhost:23306/demo?useUnicode=yes&characterEncoding=UTF-8
spring.datasource.url=jdbc:mariadb://localhost:23306/demo?useUnicode=yes&characterEncoding=UTF-8
spring.datasource.username=dev
spring.datasource.password=P@ssw0rd#
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.jpa.hibernate.naming.implicit-strategy=org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

ที่ file นี้คือส่วนของที่เราจะ main คลาสซึ่งคลาสนี้จะมีไว้สำหรับ start app
```
src/main/java/com/example/demo/DemoApplication.java
```

```java
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

```

### 4. Entity Class
เป็นส่วนที่ใช้สำหรับ mapping ข้อมูล column ใน table ของ database ให้อยู่ในรูปแบบของ Object คลาสโดยใช้ @Entity annotation เพื่อกำหนดโครงสร้างของข้อมูล Book และจะใช้ Lombok มาช่วยเพื่อลด POJO โค้ดพวก getter/setter ไป ด้วย @Data และ @Accessors โดยสร้างคลาส `Book` ดังนี้

```java
src/main/java/com/example/demo/entity/Book.java
```

ตัวอย่าง Lombok
```java
@Data
@Accessors(chain = true)
@Entity
@Table(name = "book")
public class Book {
 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "title", nullable = false, unique = true)
    private String title;

    @Column(name = "author", nullable = false)
    private String author;

}
```

ถ้าเขียนแบบ Vanilla java หละ
```java
@Entity
@Table(name = "book")
public class Book {
 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "title", nullable = false, unique = true)
    private String title;

    @Column(name = "author", nullable = false)
    private String author;

    public Long getId() {
        return this.id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return this.title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return this.author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String toString() { 
        // todo  
    }
}
```

ถ้า class ของเรามีมากกว่า 20 attribute หละเราคงต้องเขียน Getter Setter มือแหกแน่ หรือ ถึงจะใช้ tool generate Getter Setter ขึ้นมา class ของเราก็จะมีจำนวน code เยอะมาก ซึ่งถ้าเราใช้ Lombok Annotation เข้ามาช่วยนั้น มันก็จะลด code ได้ในระดับนึงเลย


### 5. Repository
เป็นส่วนของ Persistence layer หรือส่วนที่ติดต่อกับ database โดยตัวอย่างเราจะสร้าง <i>BookRepository</i> โดยเราจะต้อง extend JpaRepository interface ซึ่งใช้สำหรับจัดการ CRUD ทั้งหมด และจะเพิ่ม method findByTitle สำหรับค้นหาข้อมูลจากชื่อของ `Book` โดยกฎการตั้งชื่อจะต้องมีใน attribute ของคลาส `Book` 
```
src/main/java/com/example/demo/repository/BookRepository.java
```

```java
public interface BookRepository extends JpaRepository<Book, Long> {
    List<Book> findByTitle(String title);
}
```

### 6. Service
เป็นส่วนที่เราใช้จัดการ Business Logic ของ API เพื่อดำเนินการกับ Database ใน Persistent layer โดยจะต้องประกาศ @Service annotation ดังตัวอย่าง `BookService`

```
src/main/java/com/example/demo/service/BookService.java
```

```java
@Service
public class BookService {

    @Autowired
    BookRepository bookRepository;

    public List<Book> findAll() {
        return bookRepository.findAll();
    }

    public List<Book> findByTitle(String title) {
        return bookRepository.findByTitle(title);
    }

    public Book findOne(Long id) {
        return bookRepository.findById(id)
                .orElseThrow(BookNotFoundException::new);
    }

    public Book createBook(Book book) {
        return bookRepository.save(book);
    }

    public Book updateBook(Long id, Book book) {
        if (book.getId() != id) {
            throw new BookIdMismatchException();
        }
        bookRepository.findById(id)
                .orElseThrow(BookNotFoundException::new);
        return bookRepository.save(book);
    }

    public void deleteBook(Long id) {
        bookRepository.findById(id)
                .orElseThrow(BookNotFoundException::new);
        bookRepository.deleteById(id);
    }
}
```
โดยที่จะ @Autowired คลาส BookRepository จาก Persistence layer เข้ามาใน constructor ของคลาส และเรียกใช้ method ต่างๆ ของ Repository ดังนี้
- findAll — ค้นหาข้อมูล Book ทั้งหมดใน Repository
- findByTitle — ค้นหาข้อมูล Book ด้วยชื่อใน Repository
- findOne — ค้นหาข้อมูล Book ด้วย Id ใน Repository
- createBook — สร้างข้อมูล Book ด้วยข้อมูลที่ส่งมาเข้าไปใน Repository
- updateBook — แก้ไขข้อมูลของ Book ด้วย Id และข้อมูลที่ส่งมาใน Repository โดยเช็ค Id ของข้อมูลที่ส่งเข้ามาก่อนและตรวจสอบก่อนว่ามีข้อมูลของ Book นั้นอยู่หรือไม่
- deleteBook — ลบข้อมูลของ Book ด้วย Id ใน Repository โดยตรวจสอบก่อนว่ามีข้อมูลของ Book นั้นอยู่หรือไม่

### 7. Exception Handler
จาก Service ที่เราเขียนกันมาก่อนหน้า จะเห็นได้ว่า ในบาง method จะมีการ throw execption class ออกมา และไม่มีให้เรา import ซึ่ง class เหล่านั้นเราเขียน custom ขึ้นมาเอง เพื่อให้เราทราบว่าการ throw error แบบนี้นั้นมันมาจาก class ไหน method อะไร ซึ่งจะให้ง่ายต่อการแก้ไข code เวลาเจอบัค ซึ่งจากใน `Service` จะมี exception คลาสทั้งหมด 2 คลาส แต่ตัวอย่างจะยกตัวอย่างการเขียนคลาส `BookNotFoundException` ให้และให้ลองเขียนคลาสขึ้นมาอีกคลาสเอง
```
src/main/java/com/example/demo/exception/BookNotFoundException.java
```
```java
public class BookNotFoundException extends RuntimeException {

    public BookNotFoundException() {
        super();
    }

    public BookNotFoundException(final String message, final Throwable cause) {
        super(message, cause);
    }

    public BookNotFoundException(final String message) {
        super(message);
    }

    public BookNotFoundException(final Throwable cause) {
        super(cause);
    }
}
```

### 8. Controller

ตัวอย่างการเขียน Controller สำหรับเป็น interface ระหว่างคนที่มาร้องขอบริการของเรา โดยจะต้องประกาศ @RestController annotation ที่คลาสและเพิ่ม endpoint ต่างๆ ด้วยการ mapping กับ method ในตัวอย่างของ `BookController`
```
src/main/java/com/example/demo/api/controller/BookController.java
```

```java
@RestController
@RequestMapping("/api/books")
public class BookController {

    @Autowired
    BookService bookService;

    @GetMapping
    public List<Book> findAll() {
        return bookService.findAll();
    }

    @GetMapping("/title/{bookTitle}")
    public List<Book> findByTitle(@PathVariable String bookTitle) {
        return bookService.findByTitle(bookTitle);
    }

    @GetMapping("/{id}")
    public Book findOne(@PathVariable Long id) {
        return bookService.findOne(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Book create(@RequestBody Book book) {
        return bookService.createBook(book);
    }

    @PutMapping("/{id}")
    public Book updateBook(@RequestBody Book book, @PathVariable Long id) {
        return bookService.updateBook(id, book);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        bookService.deleteBook(id);
    }
}
```
แต่ละ method ใน BookController จะเรียกไปยัง method ของ Service layer และบาง method ตรวจสอบ return value เพื่อ return HTTP response code ที่แตกต่างกันไป

### 9. ControllerAdvice

ให้สำหรับจัดการ Exception ใน Service หรือ Controller ให้มีหน้าตาที่สวยงามหรืออ่านได้ง่ายขึ้น ให้เราใช้ 
`@RestControllerAdvice` เข้ามาช่วย handing ข้อผิดพลาดต่างๆ ของ app ที่เราพัฒนาดังตัวอย่าง
```
src/main/java/com/example/demo/api/controlleradvice/RestExceptionHandler.java
```

```java
@RestControllerAdvice
public class RestExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ BookNotFoundException.class })
    protected ResponseEntity<Object> handleNotFound(
            Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, "Book not found",
                new HttpHeaders(), HttpStatus.NOT_FOUND, request);
    }

    @ExceptionHandler({ BookIdMismatchException.class,
            ConstraintViolationException.class,
            DataIntegrityViolationException.class })
    public ResponseEntity<Object> handleBadRequest(
            Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, ex.getLocalizedMessage(),
                new HttpHeaders(), HttpStatus.BAD_REQUEST, request);
    }
    //otherwise exception
    @ExceptionHandler({ Exception.class })
    public ResponseEntity<Object> handleInternalServerError(
            Exception ex, WebRequest request) {
        return handleExceptionInternal(ex, ex.getLocalizedMessage(),
                new HttpHeaders(), HttpStatus.INTERNAL_SERVER_ERROR, request);
    }
}
```

### 10. Run Application

หลังจากเราได้ลอง implement ครบทุกส่วนแล้ว ที่นี้เราจะมาลอง start application กัน 
โดยให้เรากดที่ปุ่มสามเหลี่ยมด้านข้างที่คลาส `DemoApplication` และเลือก Run `DemoApplcation.main()` ระบบจะทำการ start application ขึ้นมาให้
![alt text](image-9.png)

ที่ console ด้านล่างจะแสดงข้อมูลต่างๆ ขิ้นมาประมาณนี้ เป็นอันว่าสามารถ start ได้เรียบร้อย

![alt text](image-10.png)

หรือลอง start ด้วยคำสั่งของ `maven` ด้วยคำสั่งดังต่อไปนี้
```
./mvnw clean install && java -jar target/demo-0.0.1-SNAPSHOT.jar
```

จากนี้เราจะทำการทดสอบเรียก API โดยจะใช้ Postman หากยังไม่มี [Download Postman](https://www.postman.com/downloads/) หากติดตั้งเรียบร้อยแล้วให้เราลองเรียกไปยัง endpoint `http://localhost:8080/api/books` HTTP POST เพื่อลองสร้างข้อมูล `Book` จาก Application ที่เราสร้างขึ้นมา

![alt text](image-4.png)

จากในรูปให้เราเพื่ม json request เข้าไปในส่วนของ Body ดังนี้
```json
{
    "title": "In the Shadows",
    "author": "Story of the year"
}
```
เมื่อทำการกดปุ่ม send ที่ Postman ระบบจะทำการสร้างข้อมูลที่เราร้องขอไปเก็บที่ database และ จะตอบกลับมาด้วย Http status 201 Created พร้อมข้อมูลที่เราสร้างไปก่อนหน้าพร้อมกับ Id กลับมาให้เรา
```json
{
    "id": 1,
    "title": "In the Shadows",
    "author": "Story of the year"
}
```
อีกตัวอย่างเป็นการร้องข้อมูล Book ทั้งหมดโดยเรียกไปยัง endpoint `http://localhost:8080/api/books` HTTP GET ระบบจะตอบกลับมาด้วย Http status 200 OK และได้ข้อมูล list ออกมา

![alt text](image-5.png)

อีกตัวอย่างเป็นการร้องข้อมูล Book ด้วย Id โดยเรียกไปยัง endpoint `http://localhost:8080/api/books/{id}` HTTP GET โดย `{id}` อันนี้เราจะแทนด้วย **1** ซึ่งเป็นข้อมูล Book ที่เราสร้างไปก่อนหน้านี้ ระบบจะตอบกลับข้อมูลกลับมา Http status 200 OK พร้อม Id กลับมาให้เรา

![alt text](image-6.png)

กรณีที่เราลองแทนค่า Id ด้วยเลขอื่นที่ยังไม่มีในระบบเมื่อทำการกด Send ระบบจะตอบกลับมาด้วย Http status 404 และตอบเป้น message กลับมาว่า `Book not found`
![alt text](image-7.png)

หลังจากนี้ให้ลอง CREATE เพิ่มเติมและลอง GET by title UPDATE และ DELETE ดูโดยใช้ HTTP METHOD ให้สัมพันธ์กับที่ระบุไว้ใน code

จากตัวอย่างข้างต้นก็เป็นตัวอย่างของการสร้าง application ด้วย spring boot ขึ้นมาหลังจากที่ลองมาแล้ว ที่นี่มาลองสร้าง Order API กัน โดยใช้ความรู้ที่กันลองมาข้างต้นมาทำ API อีกเส้นนึงสำหรับจัดการส่วนของ Order กัน

### Controller Service และ Repository จะต้องมีการจัดการข้อมูลต่างๆ โดยใช้ method ดังต่อไปนี้
- findAll
- findByOrderNumber
- findByBookId
- findById
- create
- update
- delete
