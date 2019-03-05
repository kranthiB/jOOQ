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
```
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:~/test
spring.datasource.username=sa
spring.datasource.password=
```
- Create file -  "schema.sql" (under directory - src/main/resources) , fill with below content
```
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
```
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
```
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
```
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
```
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
- In the pom.xml , add the below dependencies under <dependencies> tag
- Under the "src/main/java" , create package - "com.prokarma.config" , in this package create class - "JooqSpringBootConfiguration" and fill with the below content  
- Under the "src/main/java" , create package - "com.prokarma.service", in this package create interface - "IBookService" and fill with the below content
- Under the "src/main/java" , create package - "com.prokarma.service.impl", in this package create class - "BookServiceImpl" and fill with the below content
- Under the "src/main/java" , create package - "com.prokarma.exception", in this package create class - "ExceptionTranslator" and fill with the below content
- Under the "src/main/java" , create package - "com.prokarma.transaction", in this package create class - "JooqSpringTransaction" and fill with the below content
- Under the "src/main/java" , in the package -"com.prokarma.transaction", create class - "JooqSpringTransactionProvider" and fill with the below content
- Under the "src/test/java" , create package - "com.prokarma.jooq", in this package create class - "DBSetup" and fill with the below content
- Under the "src/test/java" , in the package -- "com.prokarma.jooq", create class - "JooqUtil" and fill with the below content    
- Under the "src/test/java" , in the package -- "com.prokarma.jooq", create class - "JooqTests" and fill with the below content    
- Under the "src/test/java" , in the package -- "com.prokarma.jooq", create class - "TransactionTest" and fill with the below content
- 
