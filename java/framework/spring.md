### 1. How to handle multiple candidate beans?
#### 1.1 @Primary
 on the implementation bean, to mark priority
 
#### 1.2 @Qualifier
use with @Autowired, on injected bean variable

#### 1.3 @Resource
@Resource(name="jdbcDeviceDao")
 use to name the specific resource for inject
 
 [Spring Injection with @Resource, @Autowired and @Inject](http://blogs.sourceallies.com/2011/08/spring-injection-with-resource-and-autowired/)