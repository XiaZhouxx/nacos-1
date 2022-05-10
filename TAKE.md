## Nacos 源码解析

### 服务端管理 

ServerMemberManager 管理Nacos服务端成员


### 服务注册
基于Nacos 2.0.4最新跨度较大的版本, 源码中还是保留了以往的版本v1, 做版本兼容 需要注意.

(ephemeral=true)NamingGrpcClientProxy -> RpcClient 长连接、InstanceRequest -> NacosServer -> GrpcBiStreamRequestAcceptor(接收长连接)/GrpcRequestAcceptor(处理请求) 
    -> InstanceRequestHandler(实例注册/注销请求处理)  

(ephemeral=false) NamingHttpClientProxy -> HTTP请求接口 /v1/ns/instance -> InstanceController
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

`2.0`之后的Grpc方式. 

1. `GrpcRequestAcceptor` 类似MVC的`DispatchServlet`, 找到对应请求的处理器(并且会更新客户端最后活跃时间, 服务端会定期检测所有客户端-健康检测/剔除 `ConnectionManager`.).
2. `InstanceRequestHandler` 负责处理实例注册/注销的处理器
3. 构建服务实例基础实体 Service = namespace + groupName + ServiceName, 构成一个服务的唯一实体.
4. 根据clientId(2.0这里实际是连接id).获取client, 存储这个客户端实体本次的服务发布信息. 
5. 发布`ClientEvent.ClientChangedEvent`, 通过`DistroProtocol`协议异步同步到集群其他节点(`DistroClientDataProcessor`)
6. 发布`ClientOperationEvent.ClientRegisterServiceEvent`, 通过`ClientServiceIndexesManager` 维护一个索引项. `k = Service,v = Set<String> clientIds`. 通过这个索引可以找到一个服务下的所有实例(长连接的client).


#### CP 方式的注册服务


#### Distro协议同步节点数据
1. `DistroProtocol`
DistroClientDataProcessor



0100 << 1 = 1000 
这里直接采用hash & oldCap的方式. 
1. 如果 == 0 说明原hash值是不含有原oldCap的进制位的, 例如`1000`、`1010`、`1011`, 而因为其掩码机制, 实际上后续的最高位都会被掩码. 那么实际有效的值只会包含在原oldCap - 1的范围内, 因此这个元素也就不需要迁移
2. 而对于 == 1的情况下. 原hash值含有原oldCap的进制位. `1100`、`1101`、`1111`、`0100`,此类因为其原oldCap - 1的掩码下. 掩盖了最高位, 但是由于扩容机制容量左移了一位, 那么 == 1的该位就会参与计算(该位换算也就是oldCap), 也就是需要迁移的curIndex + oldCap 




































