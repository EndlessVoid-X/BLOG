
## springcloud是什么？

分布式系统各模块组建之间的集成,构建分布式系统所需的“全家桶”。

- 提供服务与注册，Eureka英/juˈriːkə/
	- eureka、@EnableEurekaServer

- 网关管理,SpringCloud Gateway,Zuul（基本很少使用了）
- 配置中心，springcloud config。
	- 引入spring-cloud-config-server包
	- 注解启用@EnableConfigServer
- Hystrix 断路器
- 负载均衡，Ribbon英/ˈrɪbən/
- ，
- 消息总线、
- 数据监控、

## Eureka 原理

- 角色：Eureka server、Eureka client（provider、consumer）。
	- provider
		- Register：服务注册，提供ip+端口
		- Renew：服务续约：为什么要续约？30s，保持心跳，90s联系不上会从注册表中删除。
			- Eviction ：当 Eureka Client 和 Eureka Server 不再有心跳时，Eureka Server 会将该服务实例从服务注册列表中删除，即服务剔除。
		- Cancel：服务下线
	- consumer
		- GetRegisty：获取注册表信息。
		
- Eureka server 宕机：状态不在更新
- 通信：json/xml 压缩。
- 自我保护机制：
- 

