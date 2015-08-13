
Maven
======

The project comes with pom.xml and it is possible to compile and get the war file by following command .

mvn install


#### Configuring the data source

Database needs to be prepared to create appropriate username and password. The destination database name is medicoder. The following command configures the mysql database,

```
create database medicoder;
grant all privileges on familydoctor.* to medicoder@localhost identified by 'medicoder';
flush privileges;
```

Again the project application context should be configured with appropriate mysql username and password.

```
<bean id="dataSource"
	class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver" />
	<property name="url" value="jdbc:mysql://localhost:3306/medicoder" />
	<property name="username" value="medicoder" />
	<property name="password" value="medicoder" />
</bean>

```

If the database is not populated automatically when the site starts or if there is any login problem then the sql needs to be loaded manually. 

```
source src/main/sql/populate.sql
```

#### running the mysql server

Please refer to mysql site to run the mysql server. It is typicaly done by following command,

```
mysql.server restart
```

#### Uploaded file path

It is required to create a '/tmp' file. This file is not available in windows(C:\\tmp) most of the times. 

#### running the web server

And the site can be tested using tomcat server.

```
mvn tomcat7:run-war
```

#### creating the javadoc

Javadoc is created by the `javadoc` command.

```
mvn javadoc:javadoc
```

Javadoc is created automatically in the `target/apidocs/` directory.

#### access

The access table is given below,

user | username | pass
-----|:---------|:------
admin | admin@gmail.com | admin
doctor | doctor@gmail.com | doctor
patient | toanqc@gmail.com | toanqc





