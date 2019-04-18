1. Hytrix使用的两种方式

    1.1 通用方式整合Hytrix
    
    在Controller的方法上加上@HystrixCommand注解
    ```
    @RestController
    public class MovieController {
      private static final Logger LOGGER = LoggerFactory.getLogger(MovieController.class);
      @Autowired
      private RestTemplate restTemplate;
      @Autowired
      private LoadBalancerClient loadBalancerClient;
    
      @HystrixCommand(fallbackMethod = "findByIdFallback")
      @GetMapping("/user/{id}")
      public User findById(@PathVariable Long id) {
        return this.restTemplate.getForObject("http://microservice-provider-user/" + id, User.class);
      }
    
      public User findByIdFallback(Long id) {
        User user = new User();
        user.setId(-1L);
        user.setName("默认用户");
        return user;
      }
    
      @GetMapping("/log-user-instance")
      public void logUserInstance() {
        ServiceInstance serviceInstance = this.loadBalancerClient.choose("microservice-provider-user");
        // 打印当前选择的是哪个节点
        MovieController.LOGGER.info("{}:{}:{}", serviceInstance.getServiceId(), serviceInstance.getHost(), serviceInstance.getPort());
      }
    }
    ```

    1.2 Feign使用Hystrix
    为Feign添加回退
    ```
    /**
     * Feign的fallback测试
     * 使用@FeignClient的fallback属性指定回退类
     * @author 周立
     */
    @FeignClient(name = "microservice-provider-user", fallback = FeignClientFallback.class)
    public interface UserFeignClient {
      @RequestMapping(value = "/{id}", method = RequestMethod.GET)
      public User findById(@PathVariable("id") Long id);
    
    }
    
    /**
     * 回退类FeignClientFallback需实现Feign Client接口
     * FeignClientFallback也可以是public class，没有区别
     * @author 周立
     */
    @Component
    class FeignClientFallback implements UserFeignClient {
      @Override
      public User findById(Long id) {
        User user = new User();
        user.setId(-1L);
        user.setUsername("默认用户");
        return user;
      }
    }
    ```
    
    通过Fallback Factory检查回退原因
    ```
    @FeignClient(name = "microservice-provider-user", fallbackFactory = FeignClientFallbackFactory.class)
    public interface UserFeignClient {
      @RequestMapping(value = "/{id}", method = RequestMethod.GET)
      public User findById(@PathVariable("id") Long id);
    }
    
    /**
     * UserFeignClient的fallbackFactory类，该类需实现FallbackFactory接口，并覆写create方法
     * The fallback factory must produce instances of fallback classes that
     * implement the interface annotated by {@link FeignClient}.
     * @author 周立
     */
    @Component
    class FeignClientFallbackFactory implements FallbackFactory<UserFeignClient> {
      private static final Logger LOGGER = LoggerFactory.getLogger(FeignClientFallbackFactory.class);
    
      @Override
      public UserFeignClient create(Throwable cause) {
        return new UserFeignClient() {
          @Override
          public User findById(Long id) {
            // 日志最好放在各个fallback方法中，而不要直接放在create方法中。
            // 否则在引用启动时，就会打印该日志。
            // 详见https://github.com/spring-cloud/spring-cloud-netflix/issues/1471
            FeignClientFallbackFactory.LOGGER.info("fallback; reason was:", cause);
            User user = new User();
            user.setId(-1L);
            user.setUsername("默认用户");
            return user;
          }
        };
      }
    }
    ```
    
说明：请务必注意，在Spring Cloud Dalston中，Feign默认是不开启Hystrix的。
因此，如使用Dalston请务必额外设置属性：feign.hystrix.enabled=true，否则断路器不会生效。
而，Spring Cloud Angel/Brixton/Camden中，Feign默认都是开启Hystrix的。无需设置该属性。
