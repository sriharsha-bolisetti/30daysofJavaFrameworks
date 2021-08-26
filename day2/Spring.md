- Every Spring application also consists of such a central class, though Spring calls it `ApplicationContext`
- Spring needs a configuration class in order to construct an ApplicationContext.
- This method of configuring Spring beans is called Spring’s explicit Java configuration and it looks very similar to the application class you wrote above.
- You need to construct `ApplicationContext` from `@Configuration` class itself. Afterwards, you need to change your application to make use of this context.
- As your configuration class is annotated with Spring’s @Bean and @Configuration annotations, you’ll want to instantiate a AnnotationConfigApplicationContext. If you used Springs XML configuration file from above, you would instantiate a ClasspathXMLApplicationContext instead, leading to the same result

```
@Override
public void init() throws ServletException {
    AnnotationConfigApplicationContext ctx
            = new AnnotationConfigApplicationContext(MyFancyPdfInvoicesApplicationConfiguration.class);
    this.userService = ctx.getBean(UserService.class);
    this.objectMapper = ctx.getBean(ObjectMapper.class);
    this.invoiceService = ctx.getBean(InvoiceService.class);
}
```
- Every HTTPServlet has an init() method which you can override to do something whenever the servlet gets started. 
- Spring reads in your @Configuration class and is smart enough to construct your @beans/services, that it can give back to you whenever you call `ctx.getBean(someClass)`.
#### @Beans
- By default, Spring only constructs one instance of your  @Bean no matter how often you actually call the ctx.getBean method.
- That’s what’s called a `singleton`. And just to repeat it, by default, all @Bean methods produce singletons.
- Most Java & Spring (business) applications will make comprehensive use of singletons. If you are using singletons, however, you need to make sure that `all your singletons are thread-safe`, or else you will `run into race-conditions`.
- If you don't want to deal with that, you can also change Spring's default singleton behavior, with the `@Scope` annotation.
Eg.
```
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public UserService userService() {
    return new UserService();
}
```
- Now, instead of creating one instance of UserService, Spring will now create and return you a `completely new UserService`, every time you call `ctx.getBean()`
##### The issue with Prototypes
- Even for classes with Prototype lifecycle, singletons are only created once and will always get same Singleton back.
```
 If you are using Spring, try to go with singletons as much as possible, if you can keep them thread-safe. If not, start using prototypes, but make sure to watch out for the caveat above, i.e. when you inject them into singletons.
 ```
 -  If you want to create Spring beans from 3rd party libraries that don’t offer explicit Spring support, you will have to fall-back to @Bean methods.
 - The answer is, that @ComponentScan, by default, only scans the package and all the sub-packages of its annotated class.
 ```
 @ComponentScan(basePackageClasses = ApplicationLauncher.class)
public class MyFancyPdfInvoicesApplicationConfiguration {
```
-  You are setting the basePackageClass attribute to ApplicationLauncher. Your ApplicationLauncher lives inside the root package, and specifying it as base class will tell Spring to scan that package and all the sub-packages the class lives in.

(You could have also specified the package itself as String, via the basePackages attribute on the annotation, but that wouldn’t have been type-safe. So, if you have a class living in your package root, it is advised to use that instead.)
- Every `@SpringBootApplication` annotation is a meta annotation which includes the `@ComponentScan` annotation.
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```