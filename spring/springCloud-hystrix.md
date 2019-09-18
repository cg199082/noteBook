# springCloud组件之hystrix

参考文档：  
Hystrix系列-5-Hystrix的资源隔离策略：https://blog.csdn.net/liuchuanhong1/article/details/73718794  
SpringCloud-Hystrix(服务熔断、服务降级)：https://blog.csdn.net/www1056481167/article/details/81157171  
Hystrix面试-深入Hystrix断路器执行原理：https://blog.csdn.net/Aria_Miazzy/article/details/88070108  

使用Hystrix实现自动降级与依赖隔离[微服务]：https://www.jianshu.com/p/138f92aa83dc

# 接入方式：
1. 加入依赖包：spring-cloud-starter-hystrix
2. 启动主类添加使用@EnableCircuitBreaker注解
3. restTemplate具体函数上添加增加@HystrixCommand注解指定调用失败后的回调函数