- Frameworks like Spring Boot will pre-configure Maven to produce so called "fat jar" files. 
- They are called `fat`, because those .jar files include Tomcat and all other 3rd party libraries, all in one file. You can run them like this:
```java -jar yourjarfile.jar```
- The eay to do anything web-related in Java is through Servlet API.
- To deploy and run a servlet, a web container must be used.
- A web container is essentially the component of a web server that interacts with the servlets.
- The web container is responsible for managing the lifecycle of servlets, mapping a URL to a particular servlet and ensuring that the URL requester has the correct access rights.
##### Displaying Web pages with HttpServlets
You'll need two things to get started with servlets
1. A servlet container, something that can `run` servlets and make them available whenever you open up your browser and go to http://localhost:8080. Popular servlet containers in Java are Tomcat or Jetty.
2. The HttpServlet, containing the code to display HTML pages, itself.
Historically you had to download Tomcat onto your machine, copy your application’s .war file to the Tomcat directory and then start it.
- But in Spring, you are packaging up your application into a `.jar-file`, which means the Tomcat will live `inside` that .jar file - `you have to embed it`
- Starting up Tomcat does not automatically load your servlet, instead, it will get initialized on the very first HTTP request. If you want to start it immediately, you can set loadOnStartup to 1.
- Maven has one "tiny" inconvenience. When you build a .jar file, Maven will, `by default, only put your own source code into that .jar file`. It `will not put` the external libraries your project depends on inside that .jar file.
- To get the external libraries inside the `.jar` file you need another maven plugin - `shade plugin`

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.marcobehler.ApplicationLauncher</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```
```
<execution>
    <phase>package</phase>
    <goals>
        <goal>shade</goal>
    </goals>
```
- As soon as you call `mvn package`, the shade plugin will execute its `shade` goal, which makes sure to include all 3rd-party libraries inside your.jar file, in addition to your own source code
```
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>com.marcobehler.ApplicationLauncher</mainClass>
        </transformer>
    </transformers>
</configuration>
```
- It has to do with the way the shade plugin works. 
- The shade plugin will essentially unzip every 3rd party library and put it into your .jar file. 
- If a third-party library comes with a manifest.mf file, there’s potentially n-manifest files going into your .jar file, overwriting each other.
- If you specify the main class here, the shade plugin will make sure that your final manifest file will have the correct class specified. 
- Java doesn't offer any convinient, built-in JSON capabilities so you need a 3rd party library for that.

```
String json = new ObjectMapper().writeValueAsString(invoice);
response.getWriter().print(json);
```
- Jackson lets you annotate your fields or getters with the `@JsonProperty annotation`. 
- Its value defines what the name of the field is going to be in the resulting JSON string.
- Application class pattern - a poor man's version of dependency management.
```
public class Application {

    public static final InvoiceService invoiceService = new InvoiceService();
    public static final ObjectMapper objectMapper = new ObjectMapper();

}
```
- The class contains static final fields for the invoiceService and objectMapper, `effectively making them singletons`.
- Instead of creating the services inside your servlet, you can simply access them from the Application class.
- The major advantage being, that now any other class you are going to write can access the same services as well, also going through the Application class.
- Much better! Instead of actively getting dependencies from the Application class, `they are now "wired" together, passively`, in the Application class.

#### Spring
- It is rather simple to get started with Spring, as you only need one Maven dependency to add Spring framework to your project: `the spring-context dependency`.
