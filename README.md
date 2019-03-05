# jOOQ

jOOQ(Java Object Oriented Querying) is a persistence framework which embraces SQL.

## Features

- Building Typesafe SQL using DSL API
- Typesafe database object referencing using Code Generation.
- Easy to use API for Querying and Data fetching
- SQL logging and debugging

## Spring Boot + jOOQ Integration

- Create a project by using the site "http://start.spring.io"  and specify the dependencies - H2, JOOQ
- In "application.properties" (under directory - src/main/resources) file , add the following entries
```properties
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:~/test
spring.datasource.username=sa
spring.datasource.password=
```
- Create file -  "schema.sql" (under directory - src/main/resources) , fill with below content
```sql
DROP TABLE IF EXISTS book_to_book_store;
DROP TABLE IF EXISTS book_store;
DROP TABLE IF EXISTS book;
DROP TABLE IF EXISTS author;
  
DROP SEQUENCE IF EXISTS s_author_id;
CREATE SEQUENCE s_author_id START WITH 1;
  
CREATE TABLE author (
  id INT NOT NULL,
  first_name VARCHAR(50),
  last_name VARCHAR(50) NOT NULL,
  date_of_birth DATE,
  year_of_birth INT,
  address VARCHAR(50),
 
CONSTRAINT pk_t_author PRIMARY KEY (ID)
);
  
CREATE TABLE book (
  id INT NOT NULL,
  author_id INT NOT NULL,
  co_author_id INT,
  details_id INT,
  title VARCHAR(400) NOT NULL,
  published_in INT,
  language_id INT,
  content_text CLOB,
  content_pdf BLOB,
 
  rec_version INT,
  rec_timestamp TIMESTAMP,
 
CONSTRAINT pk_t_book PRIMARY KEY (id),
CONSTRAINT fk_t_book_author_id FOREIGN KEY (author_id) REFERENCES author(id),
CONSTRAINT fk_t_book_co_author_id FOREIGN KEY (co_author_id) REFERENCES author(id)
);
  
CREATE TABLE book_store (
  name VARCHAR(400) NOT NULL,
 
CONSTRAINT uk_t_book_store_name PRIMARY KEY(name)
);
  
CREATE TABLE book_to_book_store (
  book_store_name VARCHAR(400) NOT NULL,
  book_id INTEGER NOT NULL,
  stock INTEGER,
 
CONSTRAINT pk_b2bs PRIMARY KEY(book_store_name, book_id),
CONSTRAINT fk_b2bs_bs_name FOREIGN KEY (book_store_name)
REFERENCES book_store (name)
ON DELETE CASCADE,
CONSTRAINT fk_b2bs_b_id    FOREIGN KEY (book_id)
REFERENCES book (id)
ON DELETE CASCADE
);
```
- Create file - "data.sql" (under directory - src/main/resources) , fill with below content
```sql
INSERT INTO author VALUES (next value for s_author_id, 'George', 'Orwell', '1903-06-25', 1903, null);
INSERT INTO author VALUES (next value for s_author_id, 'Paulo', 'Coelho', '1947-08-24', 1947, null);
 
INSERT INTO book VALUES (1, 1, null, null, '1984', 1948, 1, 'To know and not to know, to be conscious of complete truthfulness while telling carefully constructed lies, to hold simultaneously two opinions which cancelled out, knowing them to be contradictory and believing in both of them, to use logic against logic, to repudiate morality while laying claim to it, to believe that democracy was impossible and that the Party was the guardian of democracy, to forget, whatever it was necessary to forget, then to draw it back into memory again at the moment when it was needed, and then promptly to forget it again, and above all, to apply the same process to the process itself -- that was the ultimate subtlety; consciously to induce unconsciousness, and then, once again, to become unconscious of the act of hypnosis you had just performed. Even to understand the word ''doublethink'' involved the use of doublethink..', null, 1, '2010-01-01 00:00:00');
INSERT INTO book VALUES (2, 1, null, null, 'Animal Farm', 1945, 1, null, null, null, '2010-01-01 00:00:00');
INSERT INTO book VALUES (3, 2, null, null, 'O Alquimista', 1988, 4, null, null, 1, null);
INSERT INTO book VALUES (4, 2, null, null, 'Brida', 1990, 2, null, null, null, null);
 
INSERT INTO book_store (name) VALUES
  ('Orell Füssli'),
   ('Ex Libris'),
   ('Buchhandlung im Volkshaus');
 
INSERT INTO book_to_book_store VALUES
  ('Orell Füssli', 1, 10),
   ('Orell Füssli', 2, 10),
   ('Orell Füssli', 3, 10),
   ('Ex Libris', 1, 1),
   ('Ex Libris', 3, 2),
   ('Buchhandlung im Volkshaus', 3, 1);
```
- In pom.xml , add the below plugin (under <build> tag) . This plugin is used to load the custom property files where
the keys present in those property files can be used in the pom configuration
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>properties-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <execution>
            <phase>initialize</phase>
            <goals>
                <goal>read-project-properties</goal>
            </goals>
            <configuration>
                <files>
                    <file>src/main/resources/application.properties</file>
                </files>
            </configuration>
        </execution>
    </executions>
</plugin>
```
- In pom.xml , add the below plugin (under <build> tag) . This plugin is used to load the schema and initialize the database with the custom scripts
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>sql-maven-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <id>create-database-h2</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>execute</goal>
            </goals>
            <configuration>
                <driver>${spring.datasource.driver-class-name}</driver>
                <url>${spring.datasource.url}</url>
                <username>${spring.datasource.username}</username>
                <password>${spring.datasource.password}</password>
                <delimiterType>row</delimiterType>
                <autocommit>true</autocommit>
                <srcFiles>
                    <srcFile>src/main/resources/schema.sql</srcFile>
                    <srcFile>src/main/resources/data.sql</srcFile>
                </srcFiles>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
</plugin>
```
- In pom.xml , add the below plugin (under <build> tag) . This plugin is used to generate the java code from the database schema.
```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <version>3.8.4</version>
    <executions>
        <execution>
            <id>generate-h2</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <jdbc>
                    <driver>${spring.datasource.driver-class-name}</driver>
                    <url>${spring.datasource.url}</url>
                    <user>${spring.datasource.username}</user>
                    <password>${spring.datasource.password}</password>
                </jdbc>
                <generator>
                    <database>
                        <name>org.jooq.util.h2.H2Database</name>
                        <includes>.*</includes>
                        <excludes></excludes>
                        <inputSchema>PUBLIC</inputSchema>
                        <recordTimestampFields>REC_TIMESTAMP</recordTimestampFields>
                    </database>
                    <generate>
                        <deprecated>false</deprecated>
                        <instanceFields>true</instanceFields>
                        <pojos>true</pojos>
                    </generate>
                    <target>
                        <packageName>com.prokarma.jooq</packageName>
                        <directory>gensrc/main/java</directory>
                    </target>
                </generator>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
</plugin>
```  
- In the above the code generated will be stored in custom directory- "gensrc". In order to add the classes present
in this directory to the classpath, add the below plugin in "pom.xml" under <build> tag
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>add-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>gensrc/main/java</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
```  
- In the pom.xml , add the below dependencies under <dependencies> tag
```xml
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>${jooq.version}</version>
</dependency>
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-meta</artifactId>
    <version>${jooq.version}</version>
</dependency>
```  
- Under the "src/main/java" , create package - "com.jooq.config" , in this package create class - "JooqSpringBootConfiguration" and fill with the below content  
```java
package com.jooq.config;
 
import com.jooq.exception.ExceptionTranslator;
import com.jooq.service.IBookService;
import com.jooq.service.impl.BookServiceImpl;
import com.jooq.transaction.JooqSpringTransactionProvider;
import org.apache.commons.dbcp.BasicDataSource;
import org.jooq.*;
import org.jooq.impl.DataSourceConnectionProvider;
import org.jooq.impl.DefaultConfiguration;
import org.jooq.impl.DefaultDSLContext;
import org.jooq.impl.DefaultExecuteListenerProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy;
 
import javax.sql.DataSource;
 
@Configuration
public class JooqSpringBootConfiguration {
 
    @Bean
    public DataSourceTransactionManager transactionManager(DataSource dataSource) {
        System.out.println(dataSource.toString());
        return new DataSourceTransactionManager(dataSource);
    }
 
    @Bean
    public DSLContext dsl(org.jooq.Configuration config) {
        return new DefaultDSLContext(config);
    }
 
    @Bean
    public ConnectionProvider connectionProvider(DataSource dataSource) {
        return new DataSourceConnectionProvider(new TransactionAwareDataSourceProxy(dataSource));
    }
 
    @Bean
    public TransactionProvider transactionProvider(DataSourceTransactionManager transactionManager) {
        return new JooqSpringTransactionProvider(transactionManager);
    }
 
    @Bean
    public ExceptionTranslator exceptionTranslator() {
        return new ExceptionTranslator();
    }
 
    @Bean
    public ExecuteListenerProvider executeListenerProvider(ExceptionTranslator exceptionTranslator) {
        return new DefaultExecuteListenerProvider(exceptionTranslator);
 
    }
 
    @Bean
    public org.jooq.Configuration jooqConfig(ConnectionProvider connectionProvider,
                                             TransactionProvider transactionProvider, ExecuteListenerProvider executeListenerProvider) {
 
        return new DefaultConfiguration()
                .derive(connectionProvider)
                .derive(transactionProvider)
                .derive(executeListenerProvider)
                .derive(SQLDialect.H2);
    }
 
    @Bean
    public IBookService bookService(DSLContext dsl) {
        return new BookServiceImpl(dsl);
    }
 
}
```
- Under the "src/main/java" , create package - "com.jooq.service", in this package create interface - "IBookService" and fill with the below content
```java
package com.jooq.service;
 
import org.springframework.transaction.annotation.Transactional;
 
public interface IBookService {
    @Transactional
    void create(int id, int authorId, String title);
}
```
- Under the "src/main/java" , create package - "com.jooq.service.impl", in this package create class - "BookServiceImpl" and fill with the below content
```java
package com.jooq.service.impl;
 
import com.jooq.service.IBookService;
import org.jooq.DSLContext;
import org.springframework.transaction.annotation.Transactional;
 
import static com.prokarma.jooq.tables.Book.BOOK;
 
public class BookServiceImpl implements IBookService {
 
    private DSLContext dsl;
 
    public BookServiceImpl(DSLContext dsl) {
        this.dsl = dsl;
    }
 
    @Override
    @Transactional
    public void create(int id, int authorId, String title) {
        dsl.insertInto(BOOK).set(BOOK.ID, id).set(BOOK.AUTHOR_ID, authorId).set(BOOK.TITLE, title).execute();
    }
}
```
- Under the "src/main/java" , create package - "com.jooq.exception", in this package create class - "ExceptionTranslator" and fill with the below content
```java
package com.jooq.exception;
 
import org.jooq.ExecuteContext;
import org.jooq.SQLDialect;
import org.jooq.impl.DefaultExecuteListener;
import org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator;
import org.springframework.jdbc.support.SQLExceptionTranslator;
import org.springframework.jdbc.support.SQLStateSQLExceptionTranslator;
 
public class ExceptionTranslator extends DefaultExecuteListener {
    @Override
    public void exception(ExecuteContext ctx) {
        if (ctx.sqlException() != null) {
            SQLDialect sqlDialect = ctx.dialect();
            SQLExceptionTranslator sqlExceptionTranslator = (sqlDialect != null)
                    ? new SQLErrorCodeSQLExceptionTranslator(sqlDialect.thirdParty().springDbName())
                    : new SQLStateSQLExceptionTranslator();
            ctx.exception(sqlExceptionTranslator.translate("jOOQ", ctx.sql(), ctx.sqlException()));
        }
    }
}
```
- Under the "src/main/java" , create package - "com.prokarma.transaction", in this package create class - "JooqSpringTransaction" and fill with the below content
```java
package com.prokarma.transaction;
 
import org.jooq.Transaction;
import org.springframework.transaction.TransactionStatus;
 
public class JooqSpringTransaction implements Transaction {
    final TransactionStatus transactionStatus;
 
    public JooqSpringTransaction(TransactionStatus transactionStatus) {
        this.transactionStatus = transactionStatus;
    }
}
```
- Under the "src/main/java" , in the package -"com.jooq.transaction", create class - "JooqSpringTransactionProvider" and fill with the below content
```java
package com.jooq.transaction;
 
import org.jooq.TransactionContext;
import org.jooq.TransactionProvider;
import org.jooq.exception.DataAccessException;
import org.jooq.tools.JooqLogger;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
 
public class JooqSpringTransactionProvider implements TransactionProvider {
 
    private static final JooqLogger JOOQ_LOGGER = JooqLogger.getLogger(JooqSpringTransactionProvider.class);
 
    private DataSourceTransactionManager dataSourceTransactionManager;
 
    public JooqSpringTransactionProvider(DataSourceTransactionManager dataSourceTransactionManager) {
        this.dataSourceTransactionManager = dataSourceTransactionManager;
    }
 
    @Override
    public void begin(TransactionContext transactionContext) throws DataAccessException {
        JOOQ_LOGGER.info("Begin Transaction");
        TransactionStatus transactionStatus = dataSourceTransactionManager.getTransaction
                (new DefaultTransactionDefinition(TransactionDefinition.PROPAGATION_NESTED));
        transactionContext.transaction(new JooqSpringTransaction(transactionStatus));
    }
 
    @Override
    public void commit(TransactionContext transactionContext) throws DataAccessException {
        JOOQ_LOGGER.info("commit transaction");
        dataSourceTransactionManager.commit(((JooqSpringTransaction) transactionContext.transaction()).transactionStatus);
    }
 
    @Override
    public void rollback(TransactionContext transactionContext) throws DataAccessException {
        JOOQ_LOGGER.info("rollback transaction");
        dataSourceTransactionManager.rollback(((JooqSpringTransaction) transactionContext.transaction()).transactionStatus);
    }
}
```
- Under the "src/test/java" , create package - "com.jooq", in this package create class - "DBSetup" and fill with the below content
```java
package com.jooq;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.util.Properties;
 
public class DBSetUp {
 
    static Connection connection;
 
    static Properties properties;
 
    public static Connection connection() {
        if (connection == null) {
            try {
                Class.forName("org.h2.Driver");
 
                connection = DriverManager.getConnection(
                        url(),
                        username(),
                        password());
                connection.setAutoCommit(false);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
 
        return connection;
    }
 
    public static String password() {
        return properties().getProperty("spring.datasource.password");
    }
 
    public static String username() {
        return properties().getProperty("spring.datasource.username");
    }
 
    public static String url() {
        return properties().getProperty("spring.datasource.url");
    }
 
    public static String driver() {
        return properties().getProperty("spring.datasource.driver-class-name");
    }
 
    public static Properties properties() {
        if (properties == null) {
            try {
                properties = new Properties();
                properties.load(DBSetUp.class.getResourceAsStream("/application.properties"));
 
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
 
        return properties;
    }
}
```
- Under the "src/test/java" , in the package -- "com.jooq", create class - "JooqUtil" and fill with the below content    
```java
package com.jooq;
 
public class JooqUtil {
 
    public static void title(String title) {
        String dashes = "=============================================================================================";
 
        System.out.println();
        System.out.println(title);
        System.out.println(dashes);
        System.out.println();
    }
 
    public static void print(Object o) {
        System.out.println(o);
    }
}
```
- Under the "src/test/java" , in the package -- "com.jooq", create class - "JooqTests" and fill with the below content    
```java
package com.jooq;
 
import static com.prokarma.jooq.tables.Author.AUTHOR;
import static com.prokarma.jooq.tables.Book.BOOK;
import static com.prokarma.jooq.tables.BookStore.BOOK_STORE;
import static com.prokarma.jooq.tables.BookToBookStore.BOOK_TO_BOOK_STORE;
import static org.jooq.impl.DSL.*;
 
import com.prokarma.jooq.tables.Author;
import com.prokarma.jooq.tables.Book;
import com.prokarma.jooq.tables.BookStore;
import com.prokarma.jooq.tables.BookToBookStore;
import com.prokarma.jooq.tables.records.AuthorRecord;
import com.prokarma.jooq.tables.records.BookRecord;
import org.apache.commons.dbcp.BasicDataSource;
import org.jooq.*;
import org.jooq.conf.*;
import org.jooq.exception.DataChangedException;
import org.jooq.impl.*;
import org.junit.Assert;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.junit4.SpringRunner;
 
import java.sql.*;
import java.util.Arrays;
 
 
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class JooqTests {
 
    Connection connection = DBSetUp.connection();
    DSLContext dslContext = DSL.using(connection);
 
    @Test
    public void test01PrintTheQuery() {
        JooqUtil.title("Create a simple query without executing it");
        JooqUtil.print(select(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
                .from(AUTHOR)
                .orderBy(AUTHOR.ID));
    }
 
    @Test
    public void test02ExecuteTheQuery() {
        JooqUtil.title("Selecting FIRST_NAME and LAST_NAME from the AUTHOR table");
        JooqUtil.print(dslContext
     .select(AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
                .from(AUTHOR)
                .orderBy(AUTHOR.ID).fetch());
    }
 
    @Test
    public void test03DMLStatements() {
        JooqUtil.title("Inserting a new AUTHOR");
        JooqUtil.print(dslContext
     .insertInto(AUTHOR, AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
                .values(3, "FIRST3", "LAST3")
                .execute());
 
        JooqUtil.title("Check if our latest record was really created");
        JooqUtil.print(dslContext
     .select()
                .from(AUTHOR)
                .where(AUTHOR.ID.eq(3))
                .fetch());
 
        JooqUtil.title("Update DATE_OF_BIRTH Column");
        JooqUtil.print(dslContext
     .update(AUTHOR)
                .set(AUTHOR.DATE_OF_BIRTH, Date.valueOf("2000-08-13"))
                .where(AUTHOR.ID.eq(3))
                .execute());
 
        JooqUtil.title("Check if our latest record was really updated");
        JooqUtil.print(dslContext
     .select()
                .from(AUTHOR)
                .where(AUTHOR.ID.eq(3))
                .fetch());
 
        JooqUtil.title("Delete the new record again");
        JooqUtil.print(dslContext
     .delete(AUTHOR)
                .where(AUTHOR.ID.eq(3))
                .execute());
 
        JooqUtil.title("Check if the record was really deleted");
        JooqUtil.print(dslContext
     .select()
                .from(AUTHOR)
                .fetch());
    }
 
    @Test
    public void test04Predicates() {
        JooqUtil.title("Combine predicates using AND");
        JooqUtil.print(dslContext
     .select()
                .from(BOOK)
                .where(BOOK.TITLE.like("%a%").and(BOOK.AUTHOR_ID.eq(1)))
                .fetch());
 
        JooqUtil.title("Use an IN-Predicate");
        JooqUtil.print(dslContext
     .select()
                .from(AUTHOR)
                .where(AUTHOR.ID.in(select(BOOK.AUTHOR_ID).from(BOOK)))
                .fetch());
    }
 
    @Test
    public void test05ColumnExpressions() {
        JooqUtil.title("CONCAT() function with prefix notation");
        JooqUtil.print(dslContext
     .select(concat(AUTHOR.FIRST_NAME, val(" "), AUTHOR.LAST_NAME))
                .from(AUTHOR)
                .orderBy(AUTHOR.ID)
                .fetch());
 
        JooqUtil.title("CONCAT() function with infix notation");
        JooqUtil.print(dslContext
     .select(AUTHOR.FIRST_NAME.concat(" ").concat(AUTHOR.LAST_NAME))
                .from(AUTHOR)
                .orderBy(AUTHOR.ID)
                .fetch());
    }
 
    @Test
    public void test06ActiveRecords() {
        AuthorRecord author;
        JooqUtil.title("Loading and changing active records");
        author = dslContext.selectFrom(AUTHOR).where(AUTHOR.ID.eq(1)).fetchOne();
        author.setDateOfBirth(Date.valueOf("2000-01-01"));
        author.store();
        JooqUtil.print(author);
 
        JooqUtil.title("Create a new active record");
        author = dslContext.newRecord(AUTHOR);
        author.setId(3);
        author.setFirstName("First3");
        author.setLastName("Last3");
        author.store();
        JooqUtil.print(author);
 
        JooqUtil.title("Refreshing an active record");
        author = dslContext.newRecord(AUTHOR);
        author.setId(3);
        author.refresh();
        JooqUtil.print(author);
 
        JooqUtil.title("Updating an active record");
        author.setDateOfBirth(Date.valueOf("2010-01-09"));
        author.store();
        JooqUtil.print(author);
 
        JooqUtil.title("Deleting an active record");
        author.delete();
        JooqUtil.print(dslContext.selectFrom(AUTHOR).fetch());
    }
 
    @Test(expected = DataChangedException.class)
    public void test07OptimisticLocking() {
        DSLContext dsl = DSL.using(connection, new Settings().withExecuteWithOptimisticLocking(Boolean.TRUE));
        JooqUtil.title("Applying optimistic locking");
 
        BookRecord book1 = dsl.selectFrom(BOOK).where(BOOK.ID.eq(1)).fetchOne();
        BookRecord book2 = dsl.selectFrom(BOOK).where(BOOK.ID.eq(1)).fetchOne();
 
        book1.setTitle("New Title");
        book1.store();
 
        book2.setTitle("Another Title");
        book2.store();
    }
 
    @Test
  public void test08CheckedExceptions() {
        JooqUtil.title("JDBC throws lots of checked exceptions where as jOOQ doesn't throw any checked exceptions");
        // These JDBC calls can throw a SQLException
 /*try (PreparedStatement stmt = connection.prepareStatement("SELECT FIRST_NAME FROM AUTHOR");
 ResultSet rs = stmt.executeQuery()) {
 while (rs.next()) {
 System.out.println(rs.getString(1));
 }
 }*/
  dslContext
  .select(AUTHOR.FIRST_NAME)
                .from(AUTHOR)
                .fetch()
                .forEach(record -> System.out.println(record.get(AUTHOR.FIRST_NAME)));
    }
 
    @Test
    public void test09ResultSets() {
        JooqUtil.title("Using jOOQ Results in foreach loops");
        for (Record record : dslContext.selectFrom(AUTHOR).fetch()) {
            System.out.println(record);
        }
 
        JooqUtil.title("Using jOOQ Results with Java 8 streams");
        dslContext
     .selectFrom(AUTHOR)
                .fetch()
                .stream()
                .flatMap(record -> Arrays.stream(record.intoArray()))
                .forEach(System.out::println);
    }
 
    @Test
   public void test10PreparedStatements() {
       /* JooqUtil.title("Distinguishing between static and prepared statements with JDBC");
 try (Statement stmt = connection.createStatement()) {
 stmt.execute("SELECT * FROM AUTHOR");
 }
 try (PreparedStatement stmt = connection.prepareStatement("SELECT * FROM AUTHOR")) {
 stmt.execute();
 }*/
 
 JooqUtil.title("Distinguishing between static and prepared statements with jOOQ");
        System.out.println(DSL.using(connection, new Settings().withStatementType(StatementType.STATIC_STATEMENT))
                .fetch("SELECT * FROM AUTHOR"));
        System.out.println(dslContext.fetch("SELECT * FROM AUTHOR"));
 
    }
 
    @Test
    public void test11StatementsAndResults() {
       /* JooqUtil.title("If you don't know whether a result set is produced with JDBC");
 try (PreparedStatement stmt = connection.prepareStatement("SELECT FIRST_NAME FROM AUTHOR")) {
 boolean moreResults = stmt.execute();
 
 do {
 if (moreResults) {
 try (ResultSet rs = stmt.getResultSet()) {
 while (rs.next()) {
 System.out.println(rs.getString(1));
 }
 }
 }
 else {
 System.out.println(stmt.getUpdateCount());
 }
 } while ((moreResults = stmt.getMoreResults()) || stmt.getUpdateCount() != -1);
 }*/
 JooqUtil.title("You always know whether a result set is produced with jOOQ, because of the type");
 
        Query query = dslContext.query("UPDATE AUTHOR SET LAST_NAME = LAST_NAME");
        System.out.println(query.execute());
 
        ResultQuery<Record> resultQuery = dslContext.resultQuery("SELECT * FROM AUTHOR");
        System.out.println(resultQuery.fetch());
    }
 
    @Test
    public void test12ConnectionProvider() {
        JooqUtil.title("Using jOOQ with a standalone connection");
        System.out.println(DSL.using(connection)
                .selectFrom(AUTHOR).fetch());
 
        JooqUtil.title("Using jOOQ with a DBCP connection pool");
        BasicDataSource basicDataSource = new BasicDataSource();
        basicDataSource.setDriverClassName(DBSetUp.driver());
        basicDataSource.setUrl(DBSetUp.url());
        basicDataSource.setUsername(DBSetUp.username());
        basicDataSource.setPassword(DBSetUp.password());
 
        System.out.println(DSL.using(basicDataSource, SQLDialect.H2)
                .selectFrom(AUTHOR).fetch());
 
    }
 
    @Test
    public void test13SQLDialect() {
        JooqUtil.title("Generate SELECT 1 FROM DUAL for all SQL dialect families");
        Arrays.stream(SQLDialect.families())
                .map(dialect -> String.format("%15s : ", dialect) + DSL.using(dialect).render(DSL.selectOne()))
                .forEach(System.out::println);
    }
 
    @Test
    public void test14Settings() {
        Select<?> select = DSL.selectFrom(AUTHOR)
                .where(AUTHOR.ID.eq(3));
 
        JooqUtil.title("A couple of settings at work - Formatting");
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderFormatted(Boolean.FALSE)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderFormatted(Boolean.TRUE)).render(select));
 
        JooqUtil.title("A couple of settings at work - Schema");
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderSchema(Boolean.FALSE)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderSchema(Boolean.TRUE)).render(select));
 
        JooqUtil.title("A couple of settings at work - Name style");
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderNameStyle(RenderNameStyle.AS_IS)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderNameStyle(RenderNameStyle.LOWER)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderNameStyle(RenderNameStyle.UPPER)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderNameStyle(RenderNameStyle.QUOTED)).render(select));
 
        JooqUtil.title("A couple of settings at work - Keyword style");
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderKeywordStyle(RenderKeywordStyle.AS_IS)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderKeywordStyle(RenderKeywordStyle.LOWER)).render(select));
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderKeywordStyle(RenderKeywordStyle.UPPER)).render(select));
 
        JooqUtil.title("A couple of settings at work - Mapping");
        System.out.println(DSL.using(SQLDialect.H2, new Settings().withRenderMapping(new RenderMapping()
                .withSchemata(new MappedSchema()
                        .withInput("PUBLIC")
                        .withOutput("test")
                        .withTables(new MappedTable()
                                .withInput("AUTHOR")
                                .withOutput("test-author"))))).render(select));
    }
 
    @Test
    public void test15ExecuteListener() {
        JooqUtil.title("Displaying execution time using a custom ExecuteListener");
        ExecuteListener executeListener = new CallbackExecuteListener()
                .onStart(ctx -> {
                    ctx.data("time", System.nanoTime());
                })
                .onEnd(ctx -> {
                    Long time = (Long) ctx.data("time");
                    System.out.println("Execution Time : " + ((System.nanoTime() - time) / 1000 / 1000.0) + "ms. Query : " + ctx.sql());
                });
        DSL.using(new DefaultConfiguration()
                .set(SQLDialect.H2)
                .set(new DefaultConnectionProvider(connection))
                .set(new DefaultExecuteListenerProvider(executeListener)))
                .selectFrom(AUTHOR)
                .fetch();
    }
 
    @Test
    public void testJoins() {
        Book b = BOOK.as("b");
        Author a = AUTHOR.as("a");
        BookStore s = BOOK_STORE.as("s");
        BookToBookStore t = BOOK_TO_BOOK_STORE.as("t");
        Result<Record3<String, String, Integer>> result = dslContext.select(a.FIRST_NAME, a.LAST_NAME, countDistinct(s.NAME))
                .from(a)
                .join(b)
                .on(b.AUTHOR_ID.equal(a.ID))
                .join(t)
                .on(t.BOOK_ID.equal(b.ID))
                .join(s)
                .on(t.BOOK_STORE_NAME.equal(s.NAME))
                .groupBy(a.FIRST_NAME, a.LAST_NAME)
                .orderBy(countDistinct(s.NAME).desc())
                .fetch();
        Assert.assertEquals(2, result.size());
        Assert.assertEquals("Paulo", result.getValue(0, a.FIRST_NAME));
        Assert.assertEquals("George", result.getValue(1, a.FIRST_NAME));
 
        Assert.assertEquals("Coelho", result.getValue(0, a.LAST_NAME));
        Assert.assertEquals("Orwell", result.getValue(1, a.LAST_NAME));
 
        Assert.assertEquals(Integer.valueOf(3), result.getValue(0, countDistinct(s.NAME)));
        Assert.assertEquals(Integer.valueOf(2), result.getValue(1, countDistinct(s.NAME)));
 
    }
 
}
```
- Under the "src/test/java" , in the package -- "com.jooq", create class - "TransactionTest" and fill with the below content
```java
package com.jooq;
 
import com.prokarma.config.JooqSpringBootConfiguration;
import com.prokarma.service.IBookService;
import org.h2.jdbc.JdbcSQLException;
import org.jooq.DSLContext;
import org.jooq.exception.DataAccessException;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
 
import java.util.concurrent.atomic.AtomicBoolean;
 
import static com.prokarma.jooq.tables.Book.BOOK;
 
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class TransactionTest {
 
    @Autowired
    DSLContext dsl;
 
    @Autowired
    DataSourceTransactionManager transactionManager;
 
    @Autowired
    IBookService bookService;
 
    @Test
    public void testExplicitTransactions() {
        boolean rollback = false;
        TransactionStatus tx = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            dsl.insertInto(BOOK).set(BOOK.ID, 1).set(BOOK.AUTHOR_ID, 1).set(BOOK.TITLE, "Book 5").execute();
            Assert.fail();
        } catch (Exception e) {
            System.out.println("Exception is" + e.toString());
            transactionManager.rollback(tx);
            rollback = true;
        }
 
        Assert.assertEquals(4, dsl.fetchCount(BOOK));
        Assert.assertTrue(rollback);
    }
 
    @Test
    public void testDeclarativeTransactions() {
        boolean rollback = false;
        try {
            bookService.create(1, 1, "Book 5");
            Assert.fail();
        } catch (Exception ignore) {
            rollback = true;
        }
 
        Assert.assertEquals(4, dsl.fetchCount(BOOK));
        Assert.assertTrue(rollback);
    }
 
    @Test
    public void testjOOQTransactionsSimple() {
        boolean rollback = false;
        try {
            dsl.transaction(c -> {
                dsl.insertInto(BOOK).set(BOOK.ID, 1).set(BOOK.AUTHOR_ID, 1).set(BOOK.TITLE, "Book 5").execute();
                Assert.fail();
            });
        } catch (Exception e) {
            rollback = true;
        }
 
        Assert.assertEquals(4, dsl.fetchCount(BOOK));
        Assert.assertTrue(rollback);
    }
 
    @Test
    public void testjOOQTransactionsNested() {
        AtomicBoolean rollback1 = new AtomicBoolean(false);
        AtomicBoolean rollback2 = new AtomicBoolean(false);
        try {
            dsl.transaction(c1 -> {
 
                dsl.insertInto(BOOK).set(BOOK.ID, 5).set(BOOK.AUTHOR_ID, 1).set(BOOK.TITLE, "Book 5").execute();
 
                Assert.assertEquals(5, dsl.fetchCount(BOOK));
 
                try {
                    dsl.transaction(c2 -> {
                        dsl.insertInto(BOOK).set(BOOK.ID, 5).set(BOOK.AUTHOR_ID, 1).set(BOOK.TITLE, "Book 6").execute();
                        Assert.fail();
                    });
                } catch (Exception e) {
                    rollback1.set(true);
                }
                Assert.assertEquals(5, dsl.fetchCount(BOOK));
 
                throw new org.jooq.exception.DataAccessException("Rollback");
            });
        } catch (org.jooq.exception.DataAccessException e) {
            Assert.assertEquals("Rollback", e.getMessage());
            rollback2.set(true);
        }
 
        Assert.assertEquals(4, dsl.fetchCount(BOOK));
        Assert.assertTrue(rollback2.get());
        Assert.assertTrue(rollback2.get());
    }
}
```
