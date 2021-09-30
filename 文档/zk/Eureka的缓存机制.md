### 缓存机制

Eureka Server 存在三个变量：(**registry、readWriteCacheMap、readOnlyCacheMap**)保存服务注册信息，默认情况下定时任务每 30s 将 readWriteCacheMap 同步至 readOnlyCacheMap，每 60s 清理超过 90s 未续约的节点，Eureka Client 每 30s 从 readOnlyCacheMap 更新服务注册信息，而 UI 则从 registry 更新服务注册信息。

拉取:先从readOnly中拉取,没有去readWrite取.再没有取实际的注册表.

下线:先更新注册表再删除readwrite.