# resurfaceio-logger-java
&copy; 2016-2018 Resurface Labs LLC

Logging usage of Java cloud apps, with user privacy by design.

## Contents

<ul>
<li><a href="#dependencies">Dependencies</a></li>
<li><a href="#installing_with_maven">Installing With Maven</a></li>
<li><a href="#logging_from_servlet_filter">Logging From Servlet Filter</a></li>
<li><a href="#logging_from_spring_boot">Logging From Spring Boot</a></li>
<li><a href="#logging_from_spark_framework">Logging From Spark Framework</a></li>
<li><a href="#logging_with_api">Logging With API</a></li>
<li><a href="#privacy">Protecting User Privacy</a></li>
</ul>

<a name="dependencies"/>

## Dependencies

Requires Java 8. No other dependencies to conflict with your app.

<a name="installing_with_maven"/>

## Installing with Maven

Add this section to `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>io.resurface</groupId>
        <artifactId>resurfaceio-logger</artifactId>
        <version>RELEASE</version>
    </dependency>
</dependencies>
```

<a name="logging_from_servlet_filter"/>

## Logging From Servlet Filter

After <a href="#installing_with_maven">installing the library</a>, add a logging filter to `web.xml`.

```xml
<filter>
    <filter-name>HttpLoggerForServlets</filter-name>
    <filter-class>io.resurface.HttpLoggerForServlets</filter-class>
    <init-param>
        <param-name>url</param-name>
        <param-value>https://...</param-value>
    </init-param>
    <init-param>
        <param-name>rules</param-name>
        <param-value>include standard</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>HttpLoggerForServlets</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

Add a CDATA section when specifying multiple rules at once like this:

```xml
    <init-param>
        <param-name>rules</param-name>
        <param-value><![CDATA[
            include standard
            sample 10
        ]]></param-value>
    </init-param>
```

<a name="logging_from_spring_boot"/>

## Logging From Spring Boot

After <a href="#installing_with_maven">installing the library</a>, configure a `FilterRegistrationBean`
to add a logging servlet filter.

```java
@Bean
public FilterRegistrationBean httpLoggerFilter() {
    FilterRegistrationBean frb = new FilterRegistrationBean();
    frb.setFilter(new io.resurface.HttpLoggerForServlets());
    frb.setName("HttpLoggerForServlets");
    frb.addUrlPatterns("/*");
    frb.addInitParameter("url", "https://...");
    frb.addInitParameter("rules", "include standard");
    return frb;
}
```

<a name="logging_from_spark_framework"/>

## Logging From Spark Framework

After <a href="#installing_with_maven">installing the library</a>, create a logger and call it from the routes of interest.

```java
import io.resurface.HttpLogger;

HttpLogger logger = new HttpLogger("https://...", "include standard");

get("/hello", (request, response) -> {
    String response_body = "Hello World";
    logger.log(request.raw(), response.raw(), response_body);
    return response_body;
});

post("/hello_post", (request, response) -> {
    String response_body = "POSTED: " + request.body();
    logger.log(request.raw(), response.raw(), response_body, request.body());
    return response_body;
});
```

Alternatively configure an `after` filter to log across multiple routes at once.

```java
after((request, response) -> {
    if (response.body() != null) {  // log successful responses only, not 404/500s
        logger.log(request.raw(), response.raw(), response.body(), request.body());
    }
});
```

<a name="logging_with_api"/>

## Logging With API

Loggers can be directly integrated into your application using our [API](API.md). This requires the most effort compared with
the options described above, but also offers the greatest flexibility and control.

[API documentation](API.md)

<a name="privacy"/>

## Protecting User Privacy

Loggers always have an active set of <a href="https://resurface.io/rules.html">rules</a> that control what data is logged
and how sensitive data is masked. All of the examples above use a standard predefined set of rules (`include standard`),
but logging rules are easily customized to meet the needs of any application.

<a href="https://resurface.io/rules.html">Logging rules documentation</a>
