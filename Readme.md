1- Objective of Example
This document is based on:
Spring Boot 2.x
Spring Security
JPA
Thymeleaf (Or JSP)
Database: MySQL, SQL Server, Oracle, Postgres
In this post, I will guide you for creating a Login application using  Spring Boot + Spring Security + JPA + Thymeleaf and explain the operating principle of Spring Security.

Users are required to log in, they can view the protected pages:

The users who have logged in the system is allowed to view only pages within their scope and role. If they access the protected pages located beyond their role, they will be denied.

See more:
Create a Login Application with Spring Boot, Spring Security, Spring JDBC
2- Prepare a Database
In the database, we have the 3 tables: APP_USER, APP_ROLE, and USER_ROLE. These are the tables in which you need to be interested. In addition, another table is PERSISTENT_LOGINS, which is used by Spring Remember Me API to store Token information, and each user's last login time.

MySQL

-- Create table
create table APP_USER
(
  USER_ID           BIGINT not null,
  USER_NAME         VARCHAR(36) not null,
  ENCRYTED_PASSWORD VARCHAR(128) not null,
  ENABLED           BIT not null 
) ;
--  
alter table APP_USER
  add constraint APP_USER_PK primary key (USER_ID);
 
alter table APP_USER
  add constraint APP_USER_UK unique (USER_NAME);
 
 
-- Create table
create table APP_ROLE
(
  ROLE_ID   BIGINT not null,
  ROLE_NAME VARCHAR(30) not null
) ;
--  
alter table APP_ROLE
  add constraint APP_ROLE_PK primary key (ROLE_ID);
 
alter table APP_ROLE
  add constraint APP_ROLE_UK unique (ROLE_NAME);
 
 
-- Create table
create table USER_ROLE
(
  ID      BIGINT not null,
  USER_ID BIGINT not null,
  ROLE_ID BIGINT not null
);
--  
alter table USER_ROLE
  add constraint USER_ROLE_PK primary key (ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_UK unique (USER_ID, ROLE_ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_FK1 foreign key (USER_ID)
  references APP_USER (USER_ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_FK2 foreign key (ROLE_ID)
  references APP_ROLE (ROLE_ID);
 
 
-- Used by Spring Remember Me API.  
CREATE TABLE Persistent_Logins (
 
    username varchar(64) not null,
    series varchar(64) not null,
    token varchar(64) not null,
    last_used timestamp not null,
    PRIMARY KEY (series)
     
);
 
--------------------------------------
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (2, 'dbuser1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (1, 'dbadmin1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
---
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (1, 'ROLE_ADMIN');
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (2, 'ROLE_USER');
 
---
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (1, 1, 1);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (2, 1, 2);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (3, 2, 2);
---
SQL Server

-- Create table
create table APP_USER
(
  USER_ID           BIGINT not null,
  USER_NAME         VARCHAR(36) not null,
  ENCRYTED_PASSWORD VARCHAR(128) not null,
  ENABLED           BIT not null 
) ;
--  
alter table APP_USER
  add constraint APP_USER_PK primary key (USER_ID);
 
alter table APP_USER
  add constraint APP_USER_UK unique (USER_NAME);
 
 
-- Create table
create table APP_ROLE
(
  ROLE_ID   BIGINT not null,
  ROLE_NAME VARCHAR(30) not null
) ;
--  
alter table APP_ROLE
  add constraint APP_ROLE_PK primary key (ROLE_ID);
 
alter table APP_ROLE
  add constraint APP_ROLE_UK unique (ROLE_NAME);
 
 
-- Create table
create table USER_ROLE
(
  ID      BIGINT not null,
  USER_ID BIGINT not null,
  ROLE_ID BIGINT not null
);
--  
alter table USER_ROLE
  add constraint USER_ROLE_PK primary key (ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_UK unique (USER_ID, ROLE_ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_FK1 foreign key (USER_ID)
  references APP_USER (USER_ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_FK2 foreign key (ROLE_ID)
  references APP_ROLE (ROLE_ID);
 
  
     
-- Used by Spring Remember Me API.  
CREATE TABLE Persistent_Logins (
 
    username varchar(64) not null,
    series varchar(64) not null,
    token varchar(64) not null,
    last_used Datetime not null,
    PRIMARY KEY (series)
     
);
 
--------------------------------------
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (2, 'dbuser1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (1, 'dbadmin1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
---
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (1, 'ROLE_ADMIN');
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (2, 'ROLE_USER');
 
---
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (1, 1, 1);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (2, 1, 2);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (3, 2, 2);
ORACLE

-- Create table
create table APP_USER
(
  USER_ID           NUMBER(19) not null,
  USER_NAME         VARCHAR2(36) not null,
  ENCRYTED_PASSWORD VARCHAR2(128) not null,
  ENABLED           NUMBER(1) not null 
) ;
--  
alter table APP_USER
  add constraint APP_USER_PK primary key (USER_ID);
  
alter table APP_USER
  add constraint APP_USER_UK unique (USER_NAME);
 
 
-- Create table
create table APP_ROLE
(
  ROLE_ID   NUMBER(19) not null,
  ROLE_NAME VARCHAR2(30) not null
) ;
--  
alter table APP_ROLE
  add constraint APP_ROLE_PK primary key (ROLE_ID);
  
alter table APP_ROLE
  add constraint APP_ROLE_UK unique (ROLE_NAME);
 
 
-- Create table
create table USER_ROLE
(
  ID      NUMBER(19) not null,
  USER_ID NUMBER(19) not null,
  ROLE_ID NUMBER(19) not null
);
--  
alter table USER_ROLE
  add constraint USER_ROLE_PK primary key (ID);
  
alter table USER_ROLE
  add constraint USER_ROLE_UK unique (USER_ID, ROLE_ID);
  
alter table USER_ROLE
  add constraint USER_ROLE_FK1 foreign key (USER_ID)
  references APP_USER (USER_ID);
  
alter table USER_ROLE
  add constraint USER_ROLE_FK2 foreign key (ROLE_ID)
  references APP_ROLE (ROLE_ID);
 
-- Used by Spring Remember Me API.  
CREATE TABLE Persistent_Logins (
 
    username varchar2(64) not null,
    series varchar2(64) not null,
    token varchar2(64) not null,
    last_used Date not null,
    PRIMARY KEY (series)
     
);
--------------------------------------
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (2, 'dbuser1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (1, 'dbadmin1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
---
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (1, 'ROLE_ADMIN');
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (2, 'ROLE_USER');
 
---
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (1, 1, 1);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (2, 1, 2);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (3, 2, 2);
---
Commit;
Postgres

-- Create table
create table APP_USER
(
  USER_ID           BIGINT not null,
  USER_NAME         VARCHAR(36) not null,
  ENCRYTED_PASSWORD VARCHAR(128) not null,
  ENABLED           Int not null 
) ;
--  
alter table APP_USER
  add constraint APP_USER_PK primary key (USER_ID);
 
alter table APP_USER
  add constraint APP_USER_UK unique (USER_NAME);
 
 
-- Create table
create table APP_ROLE
(
  ROLE_ID   BIGINT not null,
  ROLE_NAME VARCHAR(30) not null
) ;
--  
alter table APP_ROLE
  add constraint APP_ROLE_PK primary key (ROLE_ID);
 
alter table APP_ROLE
  add constraint APP_ROLE_UK unique (ROLE_NAME);
 
 
-- Create table
create table USER_ROLE
(
  ID      BIGINT not null,
  USER_ID BIGINT not null,
  ROLE_ID BIGINT not null
);
--  
alter table USER_ROLE
  add constraint USER_ROLE_PK primary key (ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_UK unique (USER_ID, ROLE_ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_FK1 foreign key (USER_ID)
  references APP_USER (USER_ID);
 
alter table USER_ROLE
  add constraint USER_ROLE_FK2 foreign key (ROLE_ID)
  references APP_ROLE (ROLE_ID);
 
  
  
-- Used by Spring Remember Me API.  
CREATE TABLE Persistent_Logins (
 
    username varchar(64) not null,
    series varchar(64) not null,
    token varchar(64) not null,
    last_used timestamp not null,
    PRIMARY KEY (series)
     
);
  
--------------------------------------
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (2, 'dbuser1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)
values (1, 'dbadmin1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);
 
---
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (1, 'ROLE_ADMIN');
 
insert into app_role (ROLE_ID, ROLE_NAME)
values (2, 'ROLE_USER');
 
---
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (1, 1, 1);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (2, 1, 2);
 
insert into user_role (ID, USER_ID, ROLE_ID)
values (3, 2, 2);
---
Commit;
3- Create a Spring Boot Project
Install Spring Tool Suite into Eclipse
On the  Eclipse, create a   Spring Boot project.

Enter:
Name: SpringBootSecurityJPA
Group: org.gauravs08
Artifact: SpringBootSecurityJPA
Description: Spring Boot +Spring Security + JPA + Remember Me.
Package: org.gauravs08.sbsecurity

In the next step, you need to select the technologies and libraries to be used (In this lesson, we will connect to Oracle, MySQL, SQL Server or  Postgres databases).
Database Libraries:
MySQL
PostgresSQL
SQL Server
Tech:
Web
Thymeleaf
Security

OK, the Project has been created.

SpringBootSecurityJpaApplication.java

package org.gauravs08.sbsecurity;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
 
@SpringBootApplication
public class SpringBootSecurityJpaApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(SpringBootSecurityJpaApplication.class, args);
    }
}
4- Configure pom.xml
If you use the Oracle database, you need to declare a necessary library to the  Oracle in the  pom.xml file:
** Oracle **

<dependencies>
    .....
 
     <dependency>
        <groupId>com.oracle</groupId>
        <artifactId>ojdbc6</artifactId>
        <version>11.2.0.3</version>
    </dependency>
     
    .....
</dependencies>
 
<repositories>
        ....
 
    <!-- Repository for ORACLE JDBC Driver -->
    <repository>
        <id>codelds</id>
        <url>https://code.lds.org/nexus/content/groups/main-repo</url>
    </repository>
     
    .....
</repositories>
If you connect to the  SQL Service database, you can use either   JTDS library or Mssql-Jdbc library:
** SQL Server **

<dependencies>
       .....
 
    <dependency>
        <groupId>com.microsoft.sqlserver</groupId>
        <artifactId>mssql-jdbc</artifactId>
        <scope>runtime</scope>
    </dependency>
     
    <dependency>
        <groupId>net.sourceforge.jtds</groupId>
        <artifactId>jtds</artifactId>
        <scope>runtime</scope>
    </dependency>
 
     .....
</dependencies>
The full content of pom.xml file:
pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
          
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>org.gauravs08</groupId>
    <artifactId>SpringBootSecurityJPA</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
 
    <name>SpringBootSecurityJPA</name>
    <description>Spring Boot +Spring Security + JPA + Remember Me</description>
 
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
 
        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>11.2.0.3</version>
        </dependency>
 
        <!-- SQL Server Mssql-Jdbc Driver -->
        <dependency>
            <groupId>com.microsoft.sqlserver</groupId>
            <artifactId>mssql-jdbc</artifactId>
            <scope>runtime</scope>
        </dependency>
 
        <!-- SQL Server JTDS Driver -->
        <dependency>
            <groupId>net.sourceforge.jtds</groupId>
            <artifactId>jtds</artifactId>
            <version>1.3.1</version>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
 
    <repositories>
        <!-- Repository for ORACLE JDBC Driver -->
        <repository>
            <id>codelds</id>
            <url>https://code.lds.org/nexus/content/groups/main-repo</url>
        </repository>
    </repositories>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
     
</project>
5- Configure Datasource
For  Spring can be connected to Database, you need configure the necessary parameters in the application.properties file.

application.properties (MySQL)

# ===============================
# DATABASE
# ===============================
 
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://tran-vmware-pc:3306/Test
spring.datasource.username=root
spring.datasource.password=12345
 
# ===============================
# JPA / HIBERNATE
# ===============================
 
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
application.properties (ORACLE)

# ===============================
# DATABASE
# ===============================
 
spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
 
spring.datasource.url=jdbc:oracle:thin:@tran-vmware-pc:1521:db12c
spring.datasource.username=Test
spring.datasource.password=12345
 
 
# ===============================
# JPA / HIBERNATE
# ===============================
 
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle10gDialect
application.properties (SQL Server + Mssql-jdbc Driver)

# ===============================
# DATABASE
# ===============================
 
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver
spring.datasource.url=jdbc:sqlserver://tran-vmware-pc\\SQLEXPRESS:1433;databaseName=Test
spring.datasource.username=sa
spring.datasource.password=12345
 
 
# ===============================
# JPA / HIBERNATE
# ===============================
 
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.SQLServer2012Dialect
application.properties (SQL Server + JTDS driver)

# ===============================
# DATABASE
# ===============================
 
spring.datasource.driver-class-name=net.sourceforge.jtds.jdbc.Driver
spring.datasource.url=jdbc:jtds:sqlserver://tran-vmware-pc:1433/Test;instance=SQLEXPRESS
spring.datasource.username=sa
spring.datasource.password=12345
 
# ===============================
# JPA / HIBERNATE
# ===============================
 
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.SQLServerDialect
application.properties (Postgres)

# ===============================
# DATABASE CONNECTION
# ===============================
 
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://tran-vmware-pc:5432/Test
spring.datasource.username=postgres
spring.datasource.password=12345
 
# ===============================
# JPA / HIBERNATE
# ===============================
 
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
 
 
# Fix Postgres JPA Error:
# Method org.postgresql.jdbc.PgConnection.createClob() is not yet implemented.
spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults=false
6- Security configuration & Remember Me
 
This application has some functions (pages) of which:
/userInfo
This is a page for viewing user's information. This page requires an user to log in and have the role such as  ROLE_ADMIN or  ROLE_USER.
/admin
This is a page for administrator. It requires users to log in, and only the people with ROLE_ADMIN role has access.
/. /welcome, /login, /logout, /403
All other pages of the application don't require users to log in.
The  WebSecurityConfig class is used to configure security for the application. It is annotated by  @Configuration. This annotation tells the  Spring that it is a configuration class and therefore, it will be analyzed by the Sping at the time when the application runs.

WebSecurityConfig.java

package org.gauravs08.sbsecurity.config;
 
import javax.sql.DataSource;
 
import org.gauravs08.sbsecurity.service.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;
import org.springframework.security.web.authentication.rememberme.PersistentTokenRepository;
 
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Autowired
    private UserDetailsServiceImpl userDetailsService;
 
    @Autowired
    private DataSource dataSource;
 
    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        return bCryptPasswordEncoder;
    }
 
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
 
        // Setting Service to find User in the database.
        // And Setting PassswordEncoder
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
 
    }
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
 
        http.csrf().disable();
 
        // The pages does not require login
        http.authorizeRequests().antMatchers("/", "/login", "/logout").permitAll();
 
        // /userInfo page requires login as ROLE_USER or ROLE_ADMIN.
        // If no login, it will redirect to /login page.
        http.authorizeRequests().antMatchers("/userInfo").access("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')");
 
        // For ADMIN only.
        http.authorizeRequests().antMatchers("/admin").access("hasRole('ROLE_ADMIN')");
 
        // When the user has logged in as XX.
        // But access a page that requires role YY,
        // AccessDeniedException will be thrown.
        http.authorizeRequests().and().exceptionHandling().accessDeniedPage("/403");
 
        // Config for Login Form
        http.authorizeRequests().and().formLogin()//
                // Submit URL of login page.
                .loginProcessingUrl("/j_spring_security_check") // Submit URL
                .loginPage("/login")//
                .defaultSuccessUrl("/userAccountInfo")//
                .failureUrl("/login?error=true")//
                .usernameParameter("username")//
                .passwordParameter("password")
                // Config for Logout Page
                .and().logout().logoutUrl("/logout").logoutSuccessUrl("/logoutSuccessful");
 
        // Config Remember Me.
        http.authorizeRequests().and() //
                .rememberMe().tokenRepository(this.persistentTokenRepository()) //
                .tokenValiditySeconds(1 * 24 * 60 * 60); // 24h
 
    }
 
    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl db = new JdbcTokenRepositoryImpl();
        db.setDataSource(dataSource);
        return db;
    }
 
}
What is "Remember Me" option?
An user accesses a website and logs in. Then he/she turns off the browser and accesses the website at some time (for example, on the next day), and he/she has to log in again, which causes unnecessary trouble. The " Remember Me" option allows the website to " remember" the user's information to automatically log in when the user visits the website the next time. 
When the user logs in an application with the " Remember Me" option, the Spring will save the last login information, and  token. The Token is an encrypted string that contains necessary information  for the  Spring to automatically log in when the user visits the website next time. 

There are two common ways for the Spring to save this information: 
Memory
Database
** WebSecurityConfig **

// Token stored in Table (Persistent_Logins)
@Bean
public PersistentTokenRepository persistentTokenRepository() {
    JdbcTokenRepositoryImpl db = new JdbcTokenRepositoryImpl();
    db.setDataSource(this.dataSource);
    return db;
}
 
// Token stored in Memory (Of Web Server).
@Bean
public PersistentTokenRepository persistentTokenRepository() {
    InMemoryTokenRepositoryImpl memory = new InMemoryTokenRepositoryImpl();
    return memory;
}
7- Entity Classes
 
AppRole.java

package org.gauravs08.sbsecurity.entity;
 
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.UniqueConstraint;
 
@Entity
@Table(name = "App_Role", //
        uniqueConstraints = { //
                @UniqueConstraint(name = "APP_ROLE_UK", columnNames = "Role_Name") })
public class AppRole {
     
    @Id
    @GeneratedValue
    @Column(name = "Role_Id", nullable = false)
    private Long roleId;
 
    @Column(name = "Role_Name", length = 30, nullable = false)
    private String roleName;
 
    public Long getRoleId() {
        return roleId;
    }
 
    public void setRoleId(Long roleId) {
        this.roleId = roleId;
    }
 
    public String getRoleName() {
        return roleName;
    }
 
    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }
     
}
AppUser.java

package org.gauravs08.sbsecurity.entity;
 
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.UniqueConstraint;
 
@Entity
@Table(name = "App_User", //
        uniqueConstraints = { //
                @UniqueConstraint(name = "APP_USER_UK", columnNames = "User_Name") })
public class AppUser {
 
    @Id
    @GeneratedValue
    @Column(name = "User_Id", nullable = false)
    private Long userId;
 
    @Column(name = "User_Name", length = 36, nullable = false)
    private String userName;
 
    @Column(name = "Encryted_Password", length = 128, nullable = false)
    private String encrytedPassword;
 
    @Column(name = "Enabled", length = 1, nullable = false)
    private boolean enabled;
 
    public Long getUserId() {
        return userId;
    }
 
    public void setUserId(Long userId) {
        this.userId = userId;
    }
 
    public String getUserName() {
        return userName;
    }
 
    public void setUserName(String userName) {
        this.userName = userName;
    }
 
    public String getEncrytedPassword() {
        return encrytedPassword;
    }
 
    public void setEncrytedPassword(String encrytedPassword) {
        this.encrytedPassword = encrytedPassword;
    }
 
    public boolean isEnabled() {
        return enabled;
    }
 
    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
 
}
UserRole.java

package org.gauravs08.sbsecurity.entity;
 
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;
import javax.persistence.Table;
import javax.persistence.UniqueConstraint;
 
@Entity
@Table(name = "User_Role", //
        uniqueConstraints = { //
                @UniqueConstraint(name = "USER_ROLE_UK", columnNames = { "User_Id", "Role_Id" }) })
public class UserRole {
 
    @Id
    @GeneratedValue
    @Column(name = "Id", nullable = false)
    private Long id;
 
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "User_Id", nullable = false)
    private AppUser appUser;
 
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "Role_Id", nullable = false)
    private AppRole appRole;
 
    public Long getId() {
        return id;
    }
 
    public void setId(Long id) {
        this.id = id;
    }
 
    public AppUser getAppUser() {
        return appUser;
    }
 
    public void setAppUser(AppUser appUser) {
        this.appUser = appUser;
    }
 
    public AppRole getAppRole() {
        return appRole;
    }
 
    public void setAppRole(AppRole appRole) {
        this.appRole = appRole;
    }
     
}
8- DAO, WebUtils
 
DAO (Data Access Object) classes are ones used to access to a database, for example, Query, Insert, Update, Delete. The  DAO classes are usually annotated by @Repository to tell the Spring "let's manage them as Spring BEANs".
The  AppUserDAO class is used to manipulate with the APP_USER table. It has a method for finding an user in the database corresponding to an username.
AppUserDAO.java

package org.gauravs08.sbsecurity.dao;
 
import javax.persistence.EntityManager;
import javax.persistence.NoResultException;
import javax.persistence.Query;
 
import org.gauravs08.sbsecurity.entity.AppUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
 
@Repository
@Transactional
public class AppUserDAO {
 
    @Autowired
    private EntityManager entityManager;
 
    public AppUser findUserAccount(String userName) {
        try {
            String sql = "Select e from " + AppUser.class.getName() + " e " //
                    + " Where e.userName = :userName ";
 
            Query query = entityManager.createQuery(sql, AppUser.class);
            query.setParameter("userName", userName);
 
            return (AppUser) query.getSingleResult();
        } catch (NoResultException e) {
            return null;
        }
    }
 
}
AppRoleDAO.java

package org.gauravs08.sbsecurity.dao;
 
import java.util.List;
 
import javax.persistence.EntityManager;
import javax.persistence.Query;
 
import org.gauravs08.sbsecurity.entity.UserRole;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
 
@Repository
@Transactional
public class AppRoleDAO {
 
    @Autowired
    private EntityManager entityManager;
 
    public List<String> getRoleNames(Long userId) {
        String sql = "Select ur.appRole.roleName from " + UserRole.class.getName() + " ur " //
                + " where ur.appUser.userId = :userId ";
 
        Query query = this.entityManager.createQuery(sql, String.class);
        query.setParameter("userId", userId);
        return query.getResultList();
    }
 
}
-
WebUtils.java

package org.gauravs08.sbsecurity.utils;
 
import java.util.Collection;
 
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;
 
public class WebUtils {
 
    public static String toString(User user) {
        StringBuilder sb = new StringBuilder();
 
        sb.append("UserName:").append(user.getUsername());
 
        Collection<GrantedAuthority> authorities = user.getAuthorities();
        if (authorities != null && !authorities.isEmpty()) {
            sb.append(" (");
            boolean first = true;
            for (GrantedAuthority a : authorities) {
                if (first) {
                    sb.append(a.getAuthority());
                    first = false;
                } else {
                    sb.append(", ").append(a.getAuthority());
                }
            }
            sb.append(")");
        }
        return sb.toString();
    }
     
}
EncrytedPasswordUtils.java

package org.gauravs08.sbsecurity.utils;
 
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
 
public class EncrytedPasswordUtils {
 
    // Encryte Password with BCryptPasswordEncoder
    public static String encrytePassword(String password) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        return encoder.encode(password);
    }
 
    public static void main(String[] args) {
        String password = "123";
        String encrytedPassword = encrytePassword(password);
 
        System.out.println("Encryted Password: " + encrytedPassword);
    }
     
}
9- UserDetailsService
UserDetailsService means a central interface in Spring Security. It is a service to search "User account and such user's roles". It is used by the  Spring Security everytime when users log in the system. Therefore, you need to write a class to implement this interface.
Herein, I create the  UserDetailsServiceImpl class which implements the UserDetailsService interface. The  UserDetailsServiceImpl class is annotated by   @Service to tell the  Spring "let's manage it as a Spring BEAN".     

UserDetailsServiceImpl.java

package org.gauravs08.sbsecurity.service;
 
import java.util.ArrayList;
import java.util.List;
 
import org.gauravs08.sbsecurity.dao.AppUserDAO;
import org.gauravs08.sbsecurity.entity.AppUser;
import org.gauravs08.sbsecurity.dao.AppRoleDAO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
 
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
 
    @Autowired
    private AppUserDAO appUserDAO;
 
    @Autowired
    private AppRoleDAO appRoleDAO;
 
    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        AppUser appUser = this.appUserDAO.findUserAccount(userName);
 
        if (appUser == null) {
            System.out.println("User not found! " + userName);
            throw new UsernameNotFoundException("User " + userName + " was not found in the database");
        }
 
        System.out.println("Found User: " + appUser);
 
        // [ROLE_USER, ROLE_ADMIN,..]
        List<String> roleNames = this.appRoleDAO.getRoleNames(appUser.getUserId());
 
        List<GrantedAuthority> grantList = new ArrayList<GrantedAuthority>();
        if (roleNames != null) {
            for (String role : roleNames) {
                // ROLE_USER, ROLE_ADMIN,..
                GrantedAuthority authority = new SimpleGrantedAuthority(role);
                grantList.add(authority);
            }
        }
 
        UserDetails userDetails = (UserDetails) new User(appUser.getUserName(), //
                appUser.getEncrytedPassword(), grantList);
 
        return userDetails;
    }
 
}
10- Controllers
MainController.java

package org.gauravs08.sbsecurity.controller;
 
import java.security.Principal;
 
import org.gauravs08.sbsecurity.utils.WebUtils;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.User;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
 
@Controller
public class MainController {
 
    @RequestMapping(value = { "/", "/welcome" }, method = RequestMethod.GET)
    public String welcomePage(Model model) {
        model.addAttribute("title", "Welcome");
        model.addAttribute("message", "This is welcome page!");
        return "welcomePage";
    }
 
    @RequestMapping(value = "/admin", method = RequestMethod.GET)
    public String adminPage(Model model, Principal principal) {
         
        User loginedUser = (User) ((Authentication) principal).getPrincipal();
 
        String userInfo = WebUtils.toString(loginedUser);
        model.addAttribute("userInfo", userInfo);
         
        return "adminPage";
    }
 
    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String loginPage(Model model) {
 
        return "loginPage";
    }
 
    @RequestMapping(value = "/logoutSuccessful", method = RequestMethod.GET)
    public String logoutSuccessfulPage(Model model) {
        model.addAttribute("title", "Logout");
        return "logoutSuccessfulPage";
    }
 
    @RequestMapping(value = "/userInfo", method = RequestMethod.GET)
    public String userInfo(Model model, Principal principal) {
 
        // After user login successfully.
        String userName = principal.getName();
 
        System.out.println("User Name: " + userName);
 
        User loginedUser = (User) ((Authentication) principal).getPrincipal();
 
        String userInfo = WebUtils.toString(loginedUser);
        model.addAttribute("userInfo", userInfo);
 
        return "userInfoPage";
    }
 
    @RequestMapping(value = "/403", method = RequestMethod.GET)
    public String accessDenied(Model model, Principal principal) {
 
        if (principal != null) {
            User loginedUser = (User) ((Authentication) principal).getPrincipal();
 
            String userInfo = WebUtils.toString(loginedUser);
 
            model.addAttribute("userInfo", userInfo);
 
            String message = "Hi " + principal.getName() //
                    + "<br> You do not have permission to access this page!";
            model.addAttribute("message", message);
 
        }
 
        return "403Page";
    }
 
}
11- Thymeleaf Template
 
The _menu.html is used as part of website. It is immersed into other sites to create the  Menu of site.
_menu.html

<div xmlns:th="http://www.thymeleaf.org"
     style="border: 1px solid #ccc;padding:5px;margin-bottom:20px;">
 
  <a th:href="@{/}">Home</a>
 
     | &nbsp;
  
   <a th:href="@{/userInfo}">User Info</a>
  
     | &nbsp;
  
   <a th:href="@{/admin}">Admin</a>
    
     | &nbsp;
      
   <a th:if="${#request.userPrincipal != null}" th:href="@{/logout}">Logout</a>
  
</div>
welcomePage.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
   <head>
      <title th:utext="${title}"></title>
   </head>
    
   <body>
    
      <!-- Include _menu.html -->
      <th:block th:include="/_menu"></th:block>  
       
      <h2>Message : <span th:utext="${message}"></span></h2>
       
   </body>
</html>
loginPage.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
   <head>
      <title>Login</title>
   </head>
   <body>
      <!-- Include _menu.html -->
      <th:block th:include="/_menu"></th:block>
       
      <h1>Login</h1>
       
      <!-- /login?error=true -->
      <div th:if="${#request.getParameter('error') == 'true'}"
            style="color:red;margin:10px 0px;">
         Login Failed!!!<br />
         Reason :
         <span th:if="${#session!= null and #session.getAttribute('SPRING_SECURITY_LAST_EXCEPTION') != null}"
            th:utext="${#session.getAttribute('SPRING_SECURITY_LAST_EXCEPTION').message}">
                Static summary
         </span>
            
      </div>
      
      <h3>Enter user name and password:</h3>
      <form name='f' th:action="@{/j_spring_security_check}" method='POST'>
         <table>
            <tr>
               <td>User:</td>
               <td><input type='text' name='username' value=''></td>
            </tr>
            <tr>
               <td>Password:</td>
               <td><input type='password' name='password' /></td>
            </tr>
            <tr>
               <td>Remember Me?</td>
               <td><input type="checkbox" name="remember-me" /></td>
            </tr>            
            <tr>
               <td><input name="submit" type="submit" value="submit" /></td>
            </tr>
         </table>
      </form>
       
      <br>
      Username/pass:
      <ul>
        <li>dbuser1/123</li>
        <li>dbadmin1/123</li>
      </ul>  
       
   </body>
    
</html>
logoutSuccessfulPage.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
   <head>
      <title>Logout</title>
   </head>
    
   <body>
      <!-- Include _menu.html -->
      <th:block th:include="/_menu"></th:block>
       
      <h1>Logout Successful!</h1>
   </body>
    
</html>
userInfoPage.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
   <head>
      <title>User Info</title>
   </head>
   <body>
      <!-- Include _menu.html -->
      <th:block th:include="/_menu"></th:block>
       
       
      <h2>User Info Page</h2>
      <h3>Welcome : <span th:utext="${#request.userPrincipal.name}"></span></h3>
      <b>This is protected page!</b>  
       
      <br/><br/>
       
      <div th:if="${userInfo != null}" th:utext="${userInfo}"></div>
       
   </body>
    
</html>
adminPage.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
   <head>
      <title th:utext="${title}"></title>
   </head>
   <body>
      <!-- Include _menu.html -->
      <th:block th:include="/_menu"></th:block>
       
      <h2>Admin Page</h2>
      <h3>Welcome :
         <span th:utext="${#request.userPrincipal.name}"></span>
      </h3>
      <b>This is protected page!</b>  
       
      <br/><br/>
       
      <div th:if="${userInfo != null}" th:utext="${userInfo}"></div>
       
   </body>
    
</html>
If users log in the system, but access a unauthorized site (not their role), the system will display the content of the /403 page to inform that the access is denied.
403Page.html

<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<title>Access Denied</title>
</head>
 
<body>
    <!-- Include _menu.html -->
    <th:block th:include="/_menu"></th:block>
 
    <h3 th:if="${message != null}" th:utext="${message}" style="color: red;"></h3>
 
    <div th:if="${userInfo != null}" th:utext="${userInfo}"></div>
     
</body>
 
</html>