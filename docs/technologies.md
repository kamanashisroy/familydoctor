Technologies
============

*Note, this page is prepared to be viewed in the github website*

#### Annotation

We used validation annotations for spring. We also used the JPA annotation like @Entity. Our pages are mapped using the @RequestMapping annotation. We used the @RequestBody to do the ajax.

#### Databinding

The databinding works for the form saving pages. It is done by @ModelAttribute annotation. It is in following controllers,

- [AdminController](../src/main/java/mum/waa/fd/app//controller/AdminController.java)
- [AppointmentController](../src/main/java/mum/waa/fd/app//controller/AppointmentController.java)
- [DoctorController](../src/main/java/mum/waa/fd/app//controller/DoctorController.java)
- [LoginController](../src/main/java/mum/waa/fd/app//controller/LoginController.java)
- [PatientController](../src/main/java/mum/waa/fd/app//controller/PatientController.java)

For example, we have it in admin page.

```
	@RequestMapping(value = "/admin/add-doctor", method = RequestMethod.GET)
	public String addDoctorAcount(@ModelAttribute("newDoctor") Doctor newDoctor, Model model) {

		model.addAttribute("specializations", appointmentService.getAllSpecialization());
		return "admin-add-doctor";
	}

```

It can be tested in all the save pages, for example doctor creation pages and patient register pages.


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

And we added multipart data in [add-doctor page](../src/main/webapp/WEB-INF/views/admin/add-doctor.jsp).

```

		<form:form commandName="newDoctor" action="${url}" method="POST" enctype="multipart/form-data" >
...
...
				<tr>
					<td class="text-align-right"><label for="picture"><spring:message
								code="label.address.picture" /></label></td>
					<td colspan="3"><form:input path="address.picture" id="picture"
							maxlength="50" size="48" type="file" /></td>
				</tr>
```

And it is added in the [Doctor class](../src/main/java/mum/waa/fd/app/domain/Doctor.java).

```
	@Transient
	private MultipartFile picture;
	private String picturePath;
```

We also added beans in the configuration file.

```
<bean id="multipartResolver"
	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="maxUploadSize" value="102400" />
</bean>
```

This feature can be viewed at the doctor add page when we log-in as admin.

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

In the right corner of every page there is a translation link to change the language of the site.

#### Exception handling

We added custom error pages instead of the exception handler. For example, the [error.jsp](../src/main/webapp/WEB-INF/views/error/error.jsp) contains the custom message.

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
        pageEncoding="UTF-8" isErrorPage="true"%>
<%@page isELIgnored="false"%>
<%@taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles"%>

<tiles:insertDefinition name="404" />
```

And there are more error pages in [error directory](../src/main/webapp/WEB-INF/views/error/).

And we configured it like the following.

```
		<security:access-denied-handler
			error-page="/403" />
```

When people hit an unknown page in the site the custom error page show displayed in the browser.

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

Security features can be seen in the home page. As the user is not logged in it asks user to login.

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

And the tiles template is in [tiles directory](../src/main/webapp/WEB-INF/tiles/). And there is a [base layout](../src/main/webapp/WEB-INF/tiles//template/baseLayout.jsp).

```
<%@ page contentType="text/html;charset=UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="tiles" uri="http://tiles.apache.org/tags-tiles"%>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title><tiles:insertAttribute name="title" /></title>
<link href="<c:url value="/resources/css/main.css" />" rel="stylesheet">
<link href="<c:url value="/resources/css/header-footer.css" />"
	rel="stylesheet">
<link rel="shortcut icon" type="image/png" sizes="128x128"
	type="image/png" href="<c:url value="/resources/images/favicon.ico" />" />

<script type="text/javascript"
	src="<spring:url value="/resources/js/lib/jquery-2.1.4.min.js" />"></script>
<script type="text/javascript"
	src="<spring:url value="/resources/js/main.js" />"></script>
</head>
<body>
	<div class="container">
		<tiles:insertAttribute name="header" />
		<tiles:insertAttribute name="body" />
		<tiles:insertAttribute name="footer" />
	</div>
</body>
</html>

```

#### Maven build

The project can be built and test using the maven build commands. Please refer to [build instruction](build_instruction.md) page.

#### Separation of concern

All our tiers are separated and they come together to render the website.

![components](https://cloud.githubusercontent.com/assets/973414/9238426/c3786380-4117-11e5-9e9b-ebd6f519a207.jpg)

