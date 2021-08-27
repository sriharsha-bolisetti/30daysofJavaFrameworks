-  You had to specifically mark these constructor dependencies with the @Autowired annotation, which you do not need anymore in newer Spring versions, as the injection is `now implicit` and as long as you don’t have multiple, unambiguous constructors which could be used to construct your object.
- The @Autowired annotation was needed (and can still be used) to signal Spring it needs to inject the correct dependencies.
- If you had multiple constructors, you would have to mark one of them with the @Autowired annotation, so Spring needs to know which one to use.
- One of the drawbacks of field injection (and you’ll learn more about this at the end of this lesson), is that `it basically hides what dependencies your class needs and you cannot easily instantiate your class outside of a Spring context anymore`.
#### @Bean lifecycles
- Sometimes you want to do something, right after bean gets created or right before the application shuts down, before the bean gets destroyed.
- Imagine your InvoiceService needs to sync itself with Amazon S3, maybe because you have a template.pdf saved onto S3 that you want to cache locally and use whenever you create an instance of your InvoiceService. Instead of downloading the template every time you want to create an invoice.
- A good way to do this, instead of doing it in the constructor, could be right after your InvoiceService gets created, but before another class can call methods on it.
##### @PostConstruct
```
@PostConstruct
public void init() {
    System.out.println("Fetching PDF Template from S3...");
    // TODO download from s3 and save locally
}
```
- if you want to make sure that your object, including all its dependencies is completely constructed, you will need to use @PostConstruct.
##### @PreDestroy
- Similarly to @PostConstruct, there’s also the @PreDestroy method, which gets called whenever you shut down your applicationContext gracefully.
- You use it `for cleaning any open resources`, like deleting that S3 template that you downloaded before with @PostConstruct, or shutting down any database connections.
- As Java does not have "destructors" like other languages, you’ll need to use that additional annotation instead.
```
@PreDestroy
public void shutdown() {
    System.out.println("Deleting downloaded templates...");
    // TODO actual deletion of PDFs
}
```
#### @PreDestroy caveat
There are two problems when using @PreDestroy.

1. The @PreDestroy methods only get executed when you shutdown the ApplicationContext. Which you usually don’t do, explicitly.
2. Most IDEs do not shut down your application gracefully, instead, they terminate it. And if you terminate your application, there’s no chance for @PreDestroys to run.
- Spring Boot automatically adds `registerShutdownHooks`
#### Environment: Resources, Properties & Profiles
- @PropertySources
```
@PropertySource("classpath:/application.properties")
//@PropertySource("classpath:/someOtherFile.properties")
public class MyFancyPdfInvoicesApplicationConfiguration {
```
- With the help of the @PropertySource annotation, Spring will try and read in properties files for you. 
- You can put in any Spring-Resources specific string here, like file:/, classpath:/ or even https:/
- You can provide multiple @PropertySources, in case you want to read in properties from multiple locations. 

#### @Profile

`@Profile("dev")`
- The DummyInvoiceServiceLoader is a normal Spring bean, but it is annotated with the @Profile annotation, which means the bean will only exist whenever you startup Spring with the dev environment 

- If you want to make sure that your Spring program does not throw an exception, if the specified file does not exist, hence you set the `ignoreResourceNotFound` flag to true.
