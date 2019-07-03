Smiley的HTTP代理Servlet
这是Java servlet形式的HTTP代理（也称为网关）。HTTP代理对于AJAX应用程序与托管Web应用程序的主机之外的主机上的Web可访问服务进行通信非常有用。它是一个反向代理，并不是真正的转发代理，尽管servlet的模板形式可能会模糊该行。

这不是第一个代理，所以我为什么要写它，为什么你会使用它呢？

这很简单 - 单个源文件实现
经过测试 - 有信心它有效 建立状态
它是安全的 - 通过Java EE web.xml或通过诸如Spring-Security之类的servlet过滤器
它是可扩展的 - 通过简单的类扩展
它可以嵌入到您的Java Web应用程序中，从而更轻松地测试您的应用程序
我已经看到很多quick'n'dirty代理在网络上以源代码形式发布，例如在博客中。我发现这样的代理支持有限的HTTP子集，例如只有GET请求，或者遇到其他实现问题，例如性能问题或URL转义错误。对这种情况感到失望，我开始创建一个运行良好且经过充分测试的简单版本，因此我知道它有效。我建议你使用经过良好测试的代理而不是未经测试的代理，这可能更好地描述为概念验证。

如果您需要比本页底部列出的更复杂的东西。

此代理依赖于Apache HttpClient，它为此代理提供了另一个扩展点。在某些时候，我可能会编写一个使用JDK的替代方案，因此没有任何依赖关系，这是可取的。同时，您必须为此及其依赖项添加jar文件：

 +- org.apache.httpcomponents:httpclient:jar:4.5.2:compile
    +- org.apache.httpcomponents:httpcore:jar:4.4.4:compile
    |  +- commons-logging:commons-logging:jar:1.2:compile
    |  \- commons-codec:commons-codec:jar:1.9:compile
此代理也支持HttpClient 4.3和更新版本。如果您需要支持较旧的 HttpClient版本，即4.1和4.2，那么请使用此代理的1.8版本。

从代理版本1.4开始，默认情况下它将识别“http.proxy”和大多数其他标准Java系统属性。

从代理版本1.5开始，可以参数化代理URL，允许您为多个目标服务器使用相同的web.xml servlet规范。它遵循 URI模板RFC，级别1。从客户端发送到ProxyServlet的特殊查询参数（请参阅下面的示例）将映射到匹配的URL模板，替换web.xml中指定的代理的targetUri中的参数。要使用它，必须使用基本servlet的子类。重要！即使使用HTTP POST，模板替换也必须放在查询字符串中。其他应用程序参数可以在POST的url-encoded-form字符串中; 只是没有proxyArgs。

构建和安装
只需在命令行使用“mvn package”构建jar。jar被构建为“target / smiley-http-proxy-servlet-VERSION.jar”。如果不修改代码，则不必构建jar，因为已发布的版本已部署到maven-central。如果您正在使用maven，那么您可以将其添加到您的pom中的依赖项中，如下所示:(注意：下面的版本不一定是最新版本。）

<dependency>
    <groupId>org.mitre.dsmiley.httpproxy</groupId>
    <artifactId>smiley-http-proxy-servlet</artifactId>
    <version>1.10</version>
</dependency>
常春藤和其他依赖管理器也可以使用。

组态
参数
以下是可配置的参数列表

log：一个布尔参数名，用于将输入和目标URL记录到servlet日志中。
forwardip：支持转发客户端IP的布尔参数名称
preserveHost：一个布尔参数名，用于保持HOST参数的原样
preserveCookies：一个布尔参数名，用于保持COOKIES的原样
http.protocol.handle-redirects：具有自动处理重定向的布尔参数名称
http.socket.timeout：用于设置套接字连接超时（毫秒）的整数参数名称
http.read.timeout：一个整数参数名，用于设置套接字读取超时（毫秒）
targetUri：要代理的目标（目标）URI的参数名称。
Servlet的
以下是与Solr服务器通信的web.xml文件的示例摘录：

<servlet>
    <servlet-name>solr</servlet-name>
    <servlet-class>org.mitre.dsmiley.httpproxy.ProxyServlet</servlet-class>
    <init-param>
      <param-name>targetUri</param-name>
      <param-value>http://solrserver:8983/solr</param-value>
    </init-param>
    <init-param>
      <param-name>log</param-name>
      <param-value>true</param-value>
    </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>solr</servlet-name>
  <url-pattern>/solr/*</url-pattern>
</servlet-mapping>
下面是一个参数化代理URL匹配查询参数_subHost，_port和_path的示例，例如“ http：// mywebapp / cluster / subpath？_subHost = namenode＆_port = 8080＆_path = monitor ”。请注意不同的代理servlet类。前导下划线不是强制性的，但在发生冲突时将它们与常规查询参数区分开来是很好的：

<servlet>
  <servlet-name>clusterProxy</servlet-name>
  <servlet-class>org.mitre.dsmiley.httpproxy.URITemplateProxyServlet</servlet-class>
  <init-param>
    <param-name>targetUri</param-name>
    <param-value>http://{_subHost}.behindfirewall.mycompany.com:{_port}/{_path}</param-value>
  </init-param>
  <init-param>
    <param-name>log</param-name>
    <param-value>true</param-value>
  </init-param>
</servlet>

<servlet-mapping>
  <servlet-name>clusterProxy</servlet-name>
  <url-pattern>/mywebapp/cluster/*</url-pattern>
</servlet-mapping>
用SpringMVC
如果你正在使用SpringMVC，那么另一种方法是使用它的 ServletWrappingController， 这样你就可以通过Spring配置这个servlet，它非常灵活，而不必修改你的web.xml。但是，请注意，可能需要一些自定义来在代理部分划分URL; 见问题＃15。

春季启动
如果您使用的是Spring Boot，请考虑以下基本配置：

@Configuration 
公共 类 SolrProxyServletConfiguration  实现 EnvironmentAware {

  @Bean 
  公共 ServletRegistrationBean  servletRegistrationBean（）{
     ServletRegistrationBean servletRegistrationBean =  新 ServletRegistrationBean（新 ProxyServlet的（），propertyResolver 。的getProperty（ “ servlet_url ”））;
    servletRegistrationBean 。addInitParameter（ProxyServlet的。 P_TARGET_URI，propertyResolver 。的getProperty（ “ target_url ”））;
    servletRegistrationBean 。addInitParameter（ProxyServlet的。 P_LOG，propertyResolver 。的getProperty（ “ logging_enabled ”，“假”））;
    return servletRegistrationBean;
  }

  私人 RelaxedPropertyResolver propertyResolver;

  @覆盖
  公共 无效 setEnvironment（环境 环境）{
     此。propertyResolver =  new  RelaxedPropertyResolver（environment，“ proxy.solr。”）;
  }
}
和属性application.yml：

proxy:
    solr:
        servlet_url: /solr/*
        target_url: http://solrserver:8983/solr
可能是Spring Boot（或Spring MVC）在servlet获取之前消耗servlet输入流的情况，这是一个问题。
请参阅问题＃83 RE禁用FilterRegistrationBean。

Dropwizard
添加Smiley对Dropwizard的代理非常简单。

在Dropwizard应用程序.yml文件中添加新属性

targetUri: http://foo.com/api  
创建新的配置属性

    @NotEmpty
    private String targetUri = "";

    @JsonProperty("targetUri")
    public String getTargetUri() {
        return targetUri;
    }  
然后通过Dropwizard服务的App run()方法使用Jetty创建注册表Smiley的代理servlet 。

@Override
    public void run(final ShepherdServiceConfiguration configuration,
        final Environment environment) {


        environment.getApplicationContext()
            .addServlet("org.mitre.dsmiley.httpproxy.ProxyServlet", "foo/*")
            .setInitParameter("targetUri", configuration.getTargetUri());  
备择方案
这个servlet有意简单，范围有限。因此，它可能无法满足您的需求，因此请考虑以下方法：

Jetty的ProxyServlet：https：//www.eclipse.org/jetty/documentation/9.4.x/proxy-servlet.html 这可能是最接近的竞争对手（简单，有限的范围，没有依赖关系），可能已经在你的类路径。
Netflix的Zuul：https：//github.com/Netflix/zuul
Charon：https：//github.com/mkopylec/charon-spring-boot-starter
