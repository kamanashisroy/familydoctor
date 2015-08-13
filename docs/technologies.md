Technologies
============

*Note, this page is prepared to be viewed in the github website*

#### Annotation

We used validation annotations for spring. We also used the JPA annotation like @entity. Our pages are mapped using the @RequestMapping annotation. We used the @RequestBody to do the ajax.

#### Databinding

The databinding works for the form saving pages. It is done by @ModelAttribute annotation.

#### Persistance in JPA 

The JPA is configured in [application context config file](src/main/webapp/WEB-INF/config/application-context.xml)


All the repositories are maintained by JPA implementation.

```
	<jpa:repositories base-package="mum.waa.fd.app.repository" />
```

There is an entity manager. It is implemented by Hibernate library.

```
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>
	<bean id="entityManagerFactory"
		class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="packagesToScan" value="mum.waa.fd.app.domain" />

		<!-- provider-specific initialization,etc. -->
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" />
		</property>
		<property name="jpaProperties">
			<props>
				<!-- validate: validate the schema, makes no changes to the database. 
					update: update the schema. create: creates the schema, destroying previous 
					data. create-drop: drop the schema at the end of the session. These options 
					intended to be developers tools and not to facilitate any production level 
					database -->
				<prop key="hibernate.hbm2ddl.auto">create-drop</prop>
				<!-- hibernate.dialect. This property makes Hibernate generate the appropriate 
					SQL for the chosen database. -->
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
				<prop key="hibernate.hbm2ddl.import_files">populate.sql</prop>
			</props>
		</property>

	</bean>
```

And data source configuration

```
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/medicoder" />
		<property name="username" value="root" />
		<property name="password" value="root" />
	</bean>

```

#### Uploading files

The file uploader packages are added in the pom file.

```
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
		</dependency>
```

#### Internationlization

Internationalization is provided by the locale configuration in the [servlet-context.xml](../src/main/webapp/WEB-INF/config/security-context.xml).

```
	<bean id="localeResolver"
                class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
                <property name="defaultLocale" value="en_US" />
        </bean>
        <mvc:interceptors>
                <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
                        <property name="paramName" value="language" />
                </bean>
        </mvc:interceptors>
```

And the translations are there in the property files.

```
	<bean id="messageSource"
		class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
		<property name="basenames" value="classpath:validation,classpath:messages" />
	</bean>
```

Suppose the english message file looks like this. This is showing [bangla language](../src/main/resources/messages_bn.properties) translation.

```
# filename messages_bn.properties
# Header
label.what.we.serve=আমাদের পরিসেবা
label.what.you.get=আপনার প্রাপ্য
label.who.we.are=আমাদের পরিচয়
label.join.us=আপনিও যোগ দিন
...
```

#### Exception handling

Exception handling is done using adviser. 

TODO fill me

#### Ajax/Rest

The ajax request is performed in the [javascript files](../src/main/webapp/resources/js/main.js).

```javascript

		ajax({
                        url : CONTEXT_ROOT + "specializations/" + spec,
                        type : "GET",
                        dataType : "json",
                        contentType : 'application/json; charset=utf-8',
                        success : function(response) {
   
                                $.each(response, function(index, value) {
                                        options += "<option value='" + index + "'>"
                                                        + value + "</options>";
                                });
        
                                $("#doctor").empty().append(options);
                        },
                        error : function() {
                                console.log("Error while request..")
                        }
                });

```

The `specializations request` is performed in the [AppointmentController](../src/main/java/mum/waa/fd/app/controller/AppointmentController.java).

```
	@RequestMapping(value = "/specializations/{spec}", method = RequestMethod.GET, consumes = "application/json", produces = {
                        "application/json" })
        public @ResponseBody Map<Integer, String> findDoctorBySpecialization(@PathVariable("spec") Specialization spec) {
                Map<Integer, String> doctorMap = doctorService.findDoctorBySpecialization(spec);
                return doctorMap;
        }

```

And it is two way, as it returns the map through json data.

#### Security 

The login data is in the [User class](../src/main/java/mum/waa/fd/app/domain/User.java).

```
	@Size(min = 5, max = 100, message = FamilyDoctorConstants.RANGE_LETTERS_VALIDATION)
        @Column(name = "PASSWORD")
        private String password;

        @Size(min = 5, max = 100, message = FamilyDoctorConstants.RANGE_LETTERS_VALIDATION)
        @Transient
        private String confirmPassword;
```

And also there is [configuration file](../src/main/webapp/WEB-INF/config/security-context.xml) that identifies the credentials by the *select* command.

```
	<security:authentication-manager>
                <security:authentication-provider>
                        <security:password-encoder hash="bcrypt" />
                        <security:jdbc-user-service
                                data-source-ref="dataSource"
                                users-by-username-query="SELECT EMAIL, PASSWORD, ENABLED FROM User WHERE EMAIL=?"
                                authorities-by-username-query="SELECT u.EMAIL, a.AUTHORITY FROM User u, AUTHORITY a WHERE u.EMAIL = a.EMAIL AND u.EMAIL=?" />
                </security:authentication-provider>
        </security:authentication-manager>
```

And the login page is configured too.

```
	<security:form-login login-page="/login"
                        login-processing-url="/doLogin" username-parameter="email"
                        password-parameter="password" default-target-url="/login-success"
                        always-use-default-target="true" authentication-failure-url="/login-failed" />
        <security:logout logout-success-url="/logout"
                        logout-url="/doLogout" />


```

#### Tiles

Tiles config is added in the [servlet-context.xml](../src/main/webapp/WEB-INF/config/servlet-context.xml).

```
	<security:form-login login-page="/login"
                        login-processing-url="/doLogin" username-parameter="email"
                        password-parameter="password" default-target-url="/login-success"
                        always-use-default-target="true" authentication-failure-url="/login-failed" />
        <security:logout logout-success-url="/logout"
                        logout-url="/doLogout" />


```

