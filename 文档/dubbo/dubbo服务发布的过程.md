我们从dubbo的demo里跟踪断点

1.我们先思考一下如果,不考虑dubbo,我们自己做服务发布,我们需要发布什么信息.

无非就是把接口和实现类以及相关的方法注入到注册中心中.

## DubboBootstrap.start

## exportServices

```java
 private void exportServices() {
   			//遍历
        configManager.getServices().forEach(sc -> {
            // TODO, compatible with ServiceConfig.export()
            ServiceConfig serviceConfig = (ServiceConfig) sc;
            serviceConfig.setBootstrap(this);
          	//是否是异步暴露
            if (exportAsync) {
              	//获取线程
                ExecutorService executor = executorRepository.getServiceExporterExecutor();
              	//异步
                Future<?> future = executor.submit(() -> {
                    sc.export();
                    exportedServices.add(sc);
                });
                asyncExportingFutures.add(future);
            } else {
              	//同步
                sc.export();
                exportedServices.add(sc);
            }
        });
    }
```

##  ServiceConfigBase.export()

抽象方法:最终由ServiceConfig去实现

```java
	//为什么要加锁呢,什么时候有并发的情况呢? 	
	public synchronized void export() {
   			//判断是否应该导出,什么时候不需要导出呢
        if (!shouldExport()) {
            return;
        }
   			//为空
        if (bootstrap == null) {
          	//获取
            bootstrap = DubboBootstrap.getInstance();
          	//初始化
            bootstrap.init();
        }
				//检查更新配置
        checkAndUpdateSubConfigs();

        //init serviceMetadata
        serviceMetadata.setVersion(version);
        serviceMetadata.setGroup(group);
        serviceMetadata.setDefaultGroup(group);
        serviceMetadata.setServiceType(getInterfaceClass());
        serviceMetadata.setServiceInterfaceName(getInterface());
        serviceMetadata.setTarget(getRef());
				//是否要延迟:核心doExport
        if (shouldDelay()) {
          	//直接用的ScheduledExecutorService
            DELAY_EXPORT_EXECUTOR.schedule(this::doExport, getDelay(), TimeUnit.MILLISECONDS);
        } else {
            doExport();
        }
    		//
        exported();
    }
```

##  doExport()

```java
 protected synchronized void doExport() {
        if (unexported) {
            throw new IllegalStateException("XXX");
        }
        if (exported) {
            return;
        }
   			//校验
        exported = true;
				//
        if (StringUtils.isEmpty(path)) {
            path = interfaceName;
        }
   			//核心
        doExportUrls();
    }
```

## doExportUrls()

```java
private void doExportUrls() {
  			//获取ServiceRepository
        ServiceRepository repository = ApplicationModel.getServiceRepository();
  			//获取服务描述
        ServiceDescriptor serviceDescriptor = repository.registerService(getInterfaceClass());
  			//先注册到ServiceRepository
        repository.registerProvider(
                getUniqueServiceName(),
                ref,
                serviceDescriptor,
                this,
                serviceMetadata
        );
				//
        List<URL> registryURLs = ConfigValidationUtils.loadRegistries(this, true);
				//
        for (ProtocolConfig protocolConfig : protocols) {
            String pathKey = URL.buildKey(getContextPath(protocolConfig)
                    .map(p -> p + "/" + path)
                    .orElse(path), group, version);
            // In case user specified path, register service one more time to map it to path.
            repository.registerService(pathKey, interfaceClass);
            // TODO, uncomment this line once service key is unified
            serviceMetadata.setServiceKey(pathKey);
            doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }
```

## doExportUrlsFor1Protocol

源码太长,复制过来格式不好看.又臭又长,看看有什么优化的思路呢?

## exportLocal

```java
 private void exportLocal(URL url) {
        URL local = URLBuilder.from(url)
                .setProtocol(LOCAL_PROTOCOL)
                .setHost(LOCALHOST_VALUE)
                .setPort(0)
                .build();
        Exporter<?> exporter = PROTOCOL.export(
                PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, local));
        exporters.add(exporter);
        logger.info("Export dubbo service " + interfaceClass.getName() + " to local registry url : " + local);
    }
```

## ProtocolListenerWrapper.export

## RegistryProtocol.export

```java
 @Override
    public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
        URL registryUrl = getRegistryUrl(originInvoker);
        // url to export locally
        URL providerUrl = getProviderUrl(originInvoker);

        // Subscribe the override data
        // FIXME When the provider subscribes, it will affect the scene : a certain JVM exposes the service and call
        //  the same service. Because the subscribed is cached key with the name of the service, it causes the
        //  subscription information to cover.
        final URL overrideSubscribeUrl = getSubscribedOverrideUrl(providerUrl);
        final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
        overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);

        providerUrl = overrideUrlWithConfig(providerUrl, overrideSubscribeListener);
        //export invoker
        final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);

        // url to registry
        final Registry registry = getRegistry(originInvoker);
        final URL registeredProviderUrl = getUrlToRegistry(providerUrl, registryUrl);

        // decide if we need to delay publish
        boolean register = providerUrl.getParameter(REGISTER_KEY, true);
        if (register) {
            register(registryUrl, registeredProviderUrl);
        }

        // register stated url on provider model
        registerStatedUrl(registryUrl, registeredProviderUrl, register);

        // Deprecated! Subscribe to override rules in 2.6.x or before.
        registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);

        exporter.setRegisterUrl(registeredProviderUrl);
        exporter.setSubscribeUrl(overrideSubscribeUrl);

        notifyExport(exporter);
        //Ensure that a new exporter instance is returned every time export
        return new DestroyableExporter<>(exporter);
    }
```











![image-20210221145030888](/Users/lumac/Library/Application Support/typora-user-images/image-20210221145030888.png)

![image-20210221145131470](/Users/lumac/Library/Application Support/typora-user-images/image-20210221145131470.png)