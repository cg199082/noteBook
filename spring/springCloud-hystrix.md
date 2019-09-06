# springCloud组件之hystrix

# 接入方式：
1. 加入依赖包：spring-cloud-starter-hystrix
2. 启动主类添加使用@EnableCircuitBreaker注解
3. restTemplate具体函数上添加增加@HystrixCommand注解指定调用失败后的回调函数