###Spring Boot执行器：Production-ready特性

Spring Boot包含很多其他的特性，它们可以帮你监控和管理发布到生产环境的应用。你可以选择使用HTTP端点，JMX或远程shell（SSH或Telnet）来管理和监控应用。审计（Auditing），健康（health）和数据采集（metrics gathering）会自动应用到你的应用。

* 开启production-ready特性

[spring-boot-actuator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator)模块提供了Spring Boot所有的production-ready特性。启用该特性的最简单方式就是添加对spring-boot-starter-actuator ‘Starter POM’的依赖。

**执行器（Actuator）的定义**：执行器是一个制造业术语，指的是用于移动或控制东西的一个机械装置。一个很小的改变就能让执行器产生大量的运动。

基于Maven的项目想要添加执行器只需添加下面的'starter'依赖：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
对于Gradle，使用下面的声明：
```java
dependencies {
    compile("org.springframework.boot:spring-boot-starter-actuator")
}
```
* 端点

执行器端点允许你监控应用及与应用进行交互。Spring Boot包含很多内置的端点，你也可以添加自己的。例如，health端点提供了应用的基本健康信息。

端点暴露的方式取决于你采用的技术类型。大部分应用选择HTTP监控，端点的ID映射到一个URL。例如，默认情况下，health端点将被映射到/health。

下面的端点都是可用的：

| ID | 描述　|敏感（Sensitive）|
| ---- | :----- | :----- |
|autoconfig|显示一个auto-configuration的报告，该报告展示所有auto-configuration候选者及它们被应用或未被应用的原因|true|
|beans|显示一个应用中所有Spring Beans的完整列表|true|
|configprops|显示一个所有@ConfigurationProperties的整理列表|true|
|dump|执行一个线程转储|true|
|env|暴露来自Spring　ConfigurableEnvironment的属性|true|
|health|展示应用的健康信息（当使用一个未认证连接访问时显示一个简单的'status'，使用认证连接访问则显示全部信息详情）|false|
|info|显示任意的应用信息|false|
|metrics|展示当前应用的'指标'信息|true|
|mappings|显示一个所有@RequestMapping路径的整理列表|true|
|shutdown|允许应用以优雅的方式关闭（默认情况下不启用）|true|
|trace|显示trace信息（默认为最新的一些HTTP请求）|true|

**注**：根据一个端点暴露的方式，sensitive参数可能会被用做一个安全提示。例如，在使用HTTP访问sensitive端点时需要提供用户名/密码（如果没有启用web安全，可能会简化为禁止访问该端点）。

1. 自定义端点

使用Spring属性可以自定义端点。你可以设置端点是否开启（enabled），是否敏感（sensitive），甚至它的id。例如，下面的application.properties改变了敏感性和beans端点的id，也启用了shutdown。
```java
endpoints.beans.id=springbeans
endpoints.beans.sensitive=false
endpoints.shutdown.enabled=true
```
**注**：前缀'endpoints + . + name'被用来唯一的标识被配置的端点。

默认情况下，除了shutdown外的所有端点都是启用的。如果希望指定选择端点的启用，你可以使用endpoints.enabled属性。例如，下面的配置禁用了除info外的所有端点：
```java
endpoints.enabled=false
endpoints.info.enabled=true
```
2. 健康信息

健康信息可以用来检查应用的运行状态。它经常被监控软件用来提醒人们生产系统是否停止。health端点暴露的默认信息取决于端点是如何被访问的。对于一个非安全，未认证的连接只返回一个简单的'status'信息。对于一个安全或认证过的连接其他详细信息也会展示（具体参考[Section 41.6, “HTTP Health endpoint access restrictions” ]()）。

健康信息是从你的ApplicationContext中定义的所有[HealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java) beans收集过来的。Spring Boot包含很多auto-configured的HealthIndicators，你也可以写自己的。

3. 安全与HealthIndicators

HealthIndicators返回的信息常常性质上有点敏感。例如，你可能不想将数据库服务器的详情发布到外面。因此，在使用一个未认证的HTTP连接时，默认只会暴露健康状态（health status）。如果想将所有的健康信息暴露出去，你可以把endpoints.health.sensitive设置为false。

为防止'拒绝服务'攻击，Health响应会被缓存。你可以使用`endpoints.health.time-to-live`属性改变默认的缓存时间（1000毫秒）。

- 自动配置的HealthIndicators

下面的HealthIndicators会被Spring Boot自动配置（在合适的时候）：

|名称|描述|
|----|:-----|
|[DiskSpaceHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/DiskSpaceHealthIndicator.java)|低磁盘空间检测|
|[DataSourceHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/DataSourceHealthIndicator.java)|检查是否能从DataSource获取连接|
|[MongoHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/MongoHealthIndicator.java)|检查一个Mongo数据库是否可用（up）|
|[RabbitHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/RabbitHealthIndicator.java)|检查一个Rabbit服务器是否可用（up）|
|[RedisHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/RedisHealthIndicator.java)|检查一个Redis服务器是否可用（up）|
|[SolrHealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/SolrHealthIndicator.java)|检查一个Solr服务器是否可用（up）|

- 编写自定义HealthIndicators

想提供自定义健康信息，你可以注册实现了[HealthIndicator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)接口的Spring beans。你需要提供一个health()方法的实现，并返回一个Health响应。Health响应需要包含一个status和可选的用于展示的详情。
```java
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealth implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}
```
除了Spring Boot预定义的[Status](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java)类型，Health也可以返回一个代表新的系统状态的自定义Status。在这种情况下，需要提供一个[HealthAggregator](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthAggregator.java)接口的自定义实现，或使用management.health.status.order属性配置默认的实现。

例如，假设一个新的，代码为FATAL的Status被用于你的一个HealthIndicator实现中。为了配置严重程度，你需要将下面的配置添加到application属性文件中：
```java
management.health.status.order: DOWN, OUT_OF_SERVICE, UNKNOWN, UP
```
如果使用HTTP访问health端点，你可能想要注册自定义的status，并使用HealthMvcEndpoint进行映射。例如，你可以将FATAL映射为HttpStatus.SERVICE_UNAVAILABLE。

4. 自定义应用info信息

通过设置Spring属性info.*，你可以定义info端点暴露的数据。所有在info关键字下的Environment属性都将被自动暴露。例如，你可以将下面的配置添加到application.properties：
```java
info.app.name=MyService
info.app.description=My awesome service
info.app.version=1.0.0
```
- 在构建时期自动扩展info属性

你可以使用已经存在的构建配置自动扩展info属性，而不是对在项目构建配置中存在的属性进行硬编码。这在Maven和Gradle都是可能的。

**使用Maven自动扩展属性**

对于Maven项目，你可以使用资源过滤来自动扩展info属性。如果使用spring-boot-starter-parent，你可以通过`@..@`占位符引用Maven的'project properties'。
```java
project.artifactId=myproject
project.name=Demo
project.version=X.X.X.X
project.description=Demo project for info endpoint
info.build.artifact=@project.artifactId@
info.build.name=@project.name@
info.build.description=@project.description@
info.build.version=@project.version@
```
**注**：在上面的示例中，我们使用project.*来设置一些值以防止由于某些原因Maven的资源过滤没有开启。Maven目标`spring-boot:run`直接将`src/main/resources`添加到classpath下（出于热加载的目的）。这就绕过了资源过滤和自动扩展属性的特性。你可以使用`exec:java`替换该目标或自定义插件的配置，具体参考[plugin usage page](http://docs.spring.io/spring-boot/docs/1.3.0.BUILD-SNAPSHOT/maven-plugin/usage.html)。

如果你不使用starter parent，在你的pom.xml你需要添加（处于<build/>元素内）：
```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```
和（处于<plugins/>内）：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
    </configuration>
</plugin>
```
**使用Gradle自动扩展属性**

通过配置Java插件的processResources任务，你也可以自动使用来自Gradle项目的属性扩展info属性。
```java
processResources {
    expand(project.properties)
}
```
然后你可以通过占位符引用Gradle项目的属性：
```java
info.build.name=${name}
info.build.description=${description}
info.build.version=${version}
```
- Git提交信息

info端点的另一个有用特性是，当项目构建完成后，它可以发布关于你的git源码仓库状态的信息。如果在你的jar中包含一个git.properties文件，git.branch和git.commit属性将被加载。

对于Maven用户，`spring-boot-starter-parent` POM包含一个能够产生git.properties文件的预配置插件。只需要简单的将下面的声明添加到你的POM中：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
对于Gradle用户可以使用一个相似的插件[gradle-git](https://github.com/ajoberstar/gradle-git)，尽管为了产生属性文件可能需要稍微多点工作。

### 基于HTTP的监控和管理

如果你正在开发一个Spring MVC应用，Spring Boot执行器自动将所有启用的端点通过HTTP暴露出去。默认约定使用端点的id作为URL路径，例如，health暴露为/health。

* 保护敏感端点

如果你的项目中添加的有Spring Security，所有通过HTTP暴露的敏感端点都会受到保护。默认情况下会使用基本认证（basic authentication，用户名为user，密码为应用启动时在控制台打印的密码）。

你可以使用Spring属性改变用户名，密码和访问端点需要的安全角色。例如，你可能会在application.properties中添加下列配置：
```java
security.user.name=admin
security.user.password=secret
management.security.role=SUPERUSER
```

**注**：如果你不使用Spring Security，那你的HTTP端点就被公开暴露，你应该慎重考虑启用哪些端点。具体参考[Section 40.1, “Customizing endpoints”](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready-customizing-endpoints)。

* 自定义管理服务器的上下文路径

有时候将所有的管理端口划分到一个路径下是有用的。例如，你的应用可能已经将`/info`作为他用。你可以用`management.contextPath`属性为管理端口设置一个前缀：
```java
management.context-path=/manage
```
上面的application.properties示例将把端口从`/{id}`改为`/manage/{id}`（比如，/manage/info）。

* 自定义管理服务器的端口

对于基于云的部署，使用默认的HTTP端口暴露管理端点（endpoints）是明智的选择。然而，如果你的应用是在自己的数据中心运行，那你可能倾向于使用一个不同的HTTP端口来暴露端点。

`management.port`属性可以用来改变HTTP端口：
```java
management.port=8081
```
由于你的管理端口经常被防火墙保护，不对外暴露也就不需要保护管理端点，即使你的主要应用是安全的。在这种情况下，classpath下会存在Spring Security库，你可以设置下面的属性来禁用安全管理策略（management security）：
```java
management.security.enabled=false
```
（如果classpath下不存在Spring Security，那也就不需要显示的以这种方式来禁用安全管理策略，它甚至可能会破坏应用程序。）

* 自定义管理服务器的地址

你可以通过设置`management.address`属性来定义管理端点可以使用的地址。这在你只想监听内部或面向生产环境的网络，或只监听来自localhost的连接时非常有用。

下面的application.properties示例不允许远程管理连接：
```java
management.port=8081
management.address=127.0.0.1
```

* 禁用HTTP端点

如果不想使用HTTP暴露端点，你可以将管理端口设置为-1：
`management.port=-1`

* HTTP Health端点访问限制

通过health端点暴露的信息根据是否为匿名访问而不同。默认情况下，当匿名访问时，任何有关服务器的健康详情都被隐藏了，该端点只简单的指示服务器是运行（up）还是停止（down）。此外，当匿名访问时，响应会被缓存一个可配置的时间段以防止端点被用于'拒绝服务'攻击。`endpoints.health.time-to-live`属性被用来配置缓存时间（单位为毫秒），默认为1000毫秒，也就是1秒。

上述的限制可以被禁止，从而允许匿名用户完全访问health端点。想达到这个效果，可以将`endpoints.health.sensitive`设为`false`。

### 基于JMX的监控和管理

Java管理扩展（JMX）提供了一种标准的监控和管理应用的机制。默认情况下，Spring Boot在`org.springframework.boot`域下将管理端点暴露为JMX MBeans。

* 自定义MBean名称

MBean的名称通常产生于端点的id。例如，health端点被暴露为`org.springframework.boot/Endpoint/HealthEndpoint`。

如果你的应用包含多个Spring ApplicationContext，你会发现存在名称冲突。为了解决这个问题，你可以将`endpoints.jmx.uniqueNames`设置为true，这样MBean的名称总是唯一的。

你也可以自定义JMX域，所有的端点都在该域下暴露。这里有个application.properties示例：
```java
endpoints.jmx.domain=myapp
endpoints.jmx.uniqueNames=true
```
* 禁用JMX端点

如果不想通过JMX暴露端点，你可以将`spring.jmx.enabled`属性设置为false：
`spring.jmx.enabled=false`

* 使用Jolokia通过HTTP实现JMX远程管理

Jolokia是一个JMX-HTTP桥，它提供了一种访问JMX beans的替代方法。想要使用Jolokia，只需添加`org.jolokia:jolokia-core`的依赖。例如，使用Maven需要添加下面的配置：
```xml
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
 </dependency>
```
在你的管理HTTP服务器上可以通过`/jolokia`访问Jolokia。

- 自定义Jolokia

Jolokia有很多配置，传统上一般使用servlet参数进行设置。使用Spring Boot，你可以在application.properties中通过把参数加上`jolokia.config.`前缀来设置：
```java
jolokia.config.debug=true
```
- 禁用Jolokia

如果你正在使用Jolokia，但不想让Spring Boot配置它，只需要简单的将`endpoints.jolokia.enabled`属性设置为false：
`endpoints.jolokia.enabled=false`   

### 使用远程shell来进行监控和管理
















