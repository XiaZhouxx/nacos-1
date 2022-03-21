## Nacos 源码解析

### 服务注册

基于Nacos 2.0.4最新跨度较大的版本, 源码中还是保留了以往的版本v1, 做版本兼容 需要注意.


#### AP 方式的注册服务(默认)
`InstanceController` 对于服务实例相关的Web Restful风格接口提供, 基于Spring MVC实现, 那么直接找到服务注册的方法
```
@PostMapping
public String register(HttpServletRequest request) throws Exception {
    // ...省略从Request中获取参数
    getInstanceOperator().registerInstance(namespaceId, serviceName, instance);
}
```
最终通过`InstanceOperatorClientImpl.registerInstance()` 


#### CP 方式的注册服务


#### Distor协议同步节点数据
DistroClientDataProcessor
