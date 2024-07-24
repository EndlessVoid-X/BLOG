# <center> springboot集成nacos
## spring clould alibaba简介

spring clould  alibaba：官网 https://sca.aliyun.com/。

github地址:https://github.com/alibaba/spring-cloud-alibaba/


spring clould alibaba 是ali开源中间件和spring clould集成。目前有4个大的分支，分别是

- 2023.0.x： https://sca.aliyun.com/docs/2023/overview/what-is-sca/
- 2022.0.x： https://sca.aliyun.com/docs/2022/overview/what-is-sca/
- 2021.0.x
- 2.2.x

## 版本说明
其每个版本都基于固定的spring cloud和springboot进行开发。并集成了Nacos Discovery，Nacos Config、Sentinel、RocketMQ、 ANS ([ANS github](https://github.com/alibaba/spring-cloud-alibaba/wiki/ANS-en "AND") )、ACM、oss、SchedulerX、 SMS等中间件。

> 我们可以在spring clould alibaba的项目pom里看到一来的spring-boot和spring-clould版本，比如https://github.com/alibaba/spring-cloud-alibaba/blob/2023.x/pom.xml，中可以看到引入的spring-cloud是`<spring.cloud.version>2023.0.3</spring.cloud.version>`，spring-boot版本是`<spring-boot.version>3.2.7</spring-boot.version>`。
> 
> 另外：spring-cloud-alibaba的版本号在为项目根目录下的`<revision>2023.0.1.2</revision>`

**所以我们得出结论是：要用spring cloud alibaba 决定了springboot 和spring cloud。你能选的是spring cloud alibaba的版本，而不是spring cloud 或者springboot版本。**


## ali官方提供的例子
在alibaba官方的例子中[（github）](https://github.com/alibaba/spring-cloud-alibaba/tree/2023.x/spring-cloud-alibaba-examples)，提供了ai-example、integrated-example、nacos-example、rocketmq-example、seata-example（seata是ali提供的分布式事务框架）、sentinel-example、spring-cloud-alibaba-sidecar-examples（边车模式，一种微服务架构）、spring-cloud-bus-rocketmq-example、spring-cloud-scheduling-example等。

本篇我们关注nacos-example（[github](https://github.com/alibaba/spring-cloud-alibaba/tree/2023.x/spring-cloud-alibaba-examples/nacos-example)）。在[readme](https://github.com/alibaba/spring-cloud-alibaba/blob/2023.x/spring-cloud-alibaba-examples/nacos-example/readme.md)中有详细的介入说明。

这里我们基于2023参考案例进行集成，以下为步骤
## 搭建本地nacos环境
## 集成实战
- step1 : 引入maven包，包括
	- spring-cloud-starter-bootstrap
	- spring-boot-starter-web
	- spring-boot-starter-actuator
	- spring-cloud-starter-alibaba-nacos-config
	- spring-cloud-starter-alibaba-nacos-discovery

	
	pom如下
	
	    <properties>
	        <!-- Project revision -->
	        <revision>2023.0.1.2</revision>
	
	        <!-- Spring Cloud -->
	        <spring.cloud.version>2023.0.3</spring.cloud.version>
	
	        <!-- Spring Boot -->
	        <spring-boot.version>3.2.7</spring-boot.version>
	
	        <!-- Maven Plugin Versions -->
	        <maven-compiler-plugin.version>3.8.1</maven-compiler-plugin.version>
	        <maven-deploy-plugin.version>2.8.2</maven-deploy-plugin.version>
	        <!-- JUnit 5 requires Surefire version 2.22.0 or higher -->
	        <maven-surefire-plugin.version>2.22.2</maven-surefire-plugin.version>
	        <maven-source-plugin.version>3.2.1</maven-source-plugin.version>
	        <maven-javadoc-plugin.version>3.2.0</maven-javadoc-plugin.version>
	        <maven-gpg-plugin.version>3.0.1</maven-gpg-plugin.version>
	        <flatten-maven-plugin.version>1.2.7</flatten-maven-plugin.version>
	        <gmavenplus-plugin.version>1.6</gmavenplus-plugin.version>
	        <jacoco.version>0.8.8</jacoco.version>
	        <native-maven-plugin.version>0.10.2</native-maven-plugin.version>
	    </properties>
	
	    <dependencyManagement>
	        <dependencies>
	
	            <!-- Spring Dependencies -->
	            <dependency>
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-dependencies</artifactId>
	                <version>${spring-boot.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	
	
	            <dependency>
	                <groupId>org.springframework.cloud</groupId>
	                <artifactId>spring-cloud-dependencies</artifactId>
	                <version>${spring.cloud.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	
	            <dependency>
	                <groupId>com.alibaba.cloud</groupId>
	                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
	                <version>2023.0.1.2</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-bootstrap</artifactId>
	            <version>3.0.3</version>
	        </dependency>
	
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba.cloud</groupId>
	            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-actuator</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.alibaba.cloud</groupId>
	            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
	        </dependency>
	
	
	    </dependencies>
	
- step2:编写启动类

		package com.example.demo;
		
		import org.springframework.boot.SpringApplication;
		import org.springframework.boot.autoconfigure.SpringBootApplication;
		import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
		import org.springframework.web.bind.annotation.GetMapping;
		import org.springframework.web.bind.annotation.PathVariable;
		import org.springframework.web.bind.annotation.RestController;
		
		
		@SpringBootApplication
		@EnableDiscoveryClient
		public class Demo1Application {
		
		    public static void main(String[] args) {
		        SpringApplication.run(Demo1Application.class, args);
		    }
		
		    @RestController
		    class EchoController {
		        @GetMapping(value = "/echo/{string}")
		        public String echo(@PathVariable String string) {
		            return string;
		        }
		    }
		}
