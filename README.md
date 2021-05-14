# java-spring-jaeger 

java-spring-jaeger 是一个日志收集器，基于 spring-boot 开发的，它不能用于生产环境，可以参考一下实现原理，它只支持 http 请求的监控。它的作用是记录进入服务的日志和服务对外请求的日志，并发送给日志监控平台，支持协议 open-tracing。

请求流程：请求 -> TracingFilter -> 业务 controller -> 调用外部 http -> tracing 拦截器（restTemplate 拦截器） -> TracingFilter。

TracingFilter 和 tracing 拦截器是它的埋点地方，通过 ThreadLocal 来进行埋点之间的追踪信息共享。

本 demo 可以快速部署一套集成 jaeger 追踪的服务群，支持 http 的请求追踪，也可测试 grpc 调用。

按下面文档执行结束后，debug 启动 notificationservice 服务，可以断点看 java-spring-jaeger 实现逻辑。

执行 `java -jar grpc-spring-boot-demo-client-1.0.jar` 向 notificationservice 发送 grpc 请求。

# Demo Letter Service

A demo application comprised of a set of Java/Spring based micro services, demonstrating Opentracing with Jaeger. Simulate dispatching of letters through a web UI to all customers who purchased a certain product and find bottlenecks in the application using the collected traces.

The services are built using Spring boot and instrumented using Jaeger's Opentracing implementation through the [spring web starter](https://github.com/opentracing-contrib/java-spring-jaeger "Spring web starter github repo") contribution.  

## Getting started

Prerequisites:

* JDK 1.8
* Maven
* Docker

First, create the docker "demo_network":

    docker network create --driver bridge demo_network 

Run the all-in-one Jagertracing docker container in the "demo_network" (refer to [Jaeger documentation](https://www.jaegertracing.io/docs/1.8/getting-started "Jaeger documentation") for more details):

    docker run -d --name jaeger -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 -p 9411:9411 --network="demo_network" jaegertracing/all-in-one
    
The Jaeger UI may be accessed on [http://localhost:16686](http://localhost:16686/ "Jaeger UI").

In order to start the notification service demo, run the following script:

    .\startAll.bat (./startAll.sh)
    
It will build the Spring applications, create Docker images and containers and start the containers. The demo application UI lives under [http://localhost:8080/index.html](http://localhost:8080/index.html "Letter service"). 
The containers will be running in the Docker bridge network "demo_network", just as the Jaegertracing applications. They will also expose their ports to the host.

In order to stop the applications and delete the containers, run:

     .\stopContainers.bat (./stopContainers.sh)
     
Commands for stopping the all-in-one Jaeger container and removing the network:

     docker stop jaeger
     docker network rm demo_network
