# Quick Start for Nacos Spring Boot Projects
<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">This quick start introduces how to enable Nacos configuration management and service discovery features for your Spring Boot project.</span></span>

<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">The quick start includes two samples:</span></span>
* How to enable dynamic configuration updates with Nacos server and nacos-config-spring-boot-starter;
* How to enable service registration and discovery with Nacos Server and nacos-discovery-spring-boot-starter.

## Prerequisite

<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Follow instructions in </span></span>[Nacos Quick Start](https://nacos.io/zh-cn/docs/quick-start.html)<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> to download Nacos and start the Nacos server.</span></span>

## Enable Configuration Service

<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Once you start the Nacos server, you can follow the steps below to enable the Nacos configuration management service for your Spring Boot project. </span></span>

Sample project: [nacos-spring-boot-config-example](https://github.com/nacos-group/nacos-examples/tree/master/nacos-spring-boot-example/nacos-spring-boot-config-example)

1. <span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Add the Nacos Spring Boot dependency.</span></span>

```
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>${latest.version}</version>
</dependency>
```

2. Configure the Nacos server address in `application.properties` :

```
nacos.config.server-addr=127.0.0.1:8848
```

3. <span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Use </span></span>`@NacosPropertySource`<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> to load the configuration source whose </span></span>`dataId`<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> is </span></span>`example`<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> , and enable autorefresh of configuration updates:</span></span>

```plain
@SpringBootApplication
@NacosPropertySource(dataId = "example", autoRefreshed = true)
public class NacosConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConfigApplication.class, args);
    }
}
```

4. <span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Specify the property value of the </span></span>`@Value`<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> annotation of Spring.</span></span>

<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>Note: </strong></span></span><span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">You need to use the  </span></span>`Setter`<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> method to enable autorefresh of configuration updates. </span></span>

```
@Controller
@RequestMapping("config")
public class ConfigController {

    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    public void setUseLocalCache(boolean useLocalCache) {
        this.useLocalCache = useLocalCache;
    }

    @RequestMapping(value = "/get", method = GET)
    @ResponseBody
    public boolean get() {
        return useLocalCache;
    }
}
```

5. Start `NacosConfigApplication`and call `curl http://localhost:8080/config/get`. You will get a return message of `false`, as no configuration has been published so far.

6. Call [Nacos Open API](https://nacos.io/zh-cn/docs/open-API.html) to publish a configuration to the Nacos server. Assume the dataId is `example`, and the content is `useLocalCache=true`.

```
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=example&group=DEFAULT_GROUP&content=useLocalCache=true"
```

7. Access `http://localhost:8080/config/get`again, and the returned value will be`true`，indicating that the value of `useLocalCache`in your application has been updated.

## Enable Service Discovery

<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Now you would also like to enable the service discovery feature of Nacos in your Spring Boot project. </span></span>

Sample project<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">: </span></span>[nacos-spring-boot-discovery-example](https://github.com/nacos-group/nacos-examples/tree/master/nacos-spring-boot-example/nacos-spring-boot-discovery-example)

1. <span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Add the Nacos Spring Boot dependency.</span></span>

```
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-discovery-spring-boot-starter</artifactId>
    <version>${latest.version}</version>
</dependency>
```

2. Configure the Nacos server address in `application.properties` :

```
nacos.discovery.server-addr=127.0.0.1:8848
```

3. Use `@NacosInjected` to inject a Nacos `NamingService` instance:

```
@Controller
@RequestMapping("discovery")
public class DiscoveryController {

    @NacosInjected
    private NamingService namingService;

    @RequestMapping(value = "/get", method = GET)
    @ResponseBody
    public List<Instance> get(@RequestParam String serviceName) throws NacosException {
        return namingService.getAllInstances(serviceName);
    }
}

@SpringBootApplication
public class NacosDiscoveryApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosDiscoveryApplication.class, args);
    }
}
```


4. Start `NacosDiscoveryApplication`and call `curl http://localhost:8080/discovery/get?serviceName=example`，you will get a return value of an empty JSON array `[]`.

5. Call [Nacos Open API](https://nacos.io/zh-cn/docs/open-API.html) to register a service called `example` to the Nacos server.

```
curl -X PUT 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=example&ip=127.0.0.1&port=8080'
```

6. Access `curl http://localhost:8080/discovery/get?serviceName=example`again and you will get the following return:

```
[
  {
    "instanceId": "127.0.0.1-8080-DEFAULT-example",
    "ip": "127.0.0.1",
    "port": 8080,
    "weight": 1.0,
    "healthy": true,
    "cluster": {
      "serviceName": null,
      "name": "",
      "healthChecker": {
        "type": "TCP"
      },
      "defaultPort": 80,
      "defaultCheckPort": 80,
      "useIPPort4Check": true,
      "metadata": {}
    },
    "service": null,
    "metadata": {}
  }
]
```

## Related Projects
* [Nacos](https://github.com/alibaba/nacos)
* [Nacos Spring](https://github.com/nacos-group/nacos-spring-project)
* [Nacos Spring Boot](https://github.com/nacos-group/nacos-spring-boot-project)
* [Spring Cloud](https://github.com/spring-cloud-incubator/spring-cloud-alibaba) [Alibaba](https://github.com/spring-cloud-incubator/spring-cloud-alibaba)
    

