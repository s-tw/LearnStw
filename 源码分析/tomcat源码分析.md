**注：这里用的tomcat版本是tomcat8.5**

本文主要是通过分析tomcat8.5源码，来来详细了解tomcat的执行过程，以及分析中间涉及到主要的部件。

在文中还会简单比较tomcat7中和tomcat8.5的区别。

由于水平有限，更多的是分析执行的流程。很多细节可能未能分析到。

注：本篇的分析是我源码阅读时的记录，相对简略。

在[tomcat源码分析](https://blog.csdn.net/weixin_47327232/category_10948734.html)中针对tomcat的执行步骤，对每个组件的执行流程做了一个更为详细的叙述。

## bootstrap 的main执行流程

* bootstrap.init();

  * initClassLoaders()；

    （初始化类加载器：commonLoader，catalinaLoader，sharedLoader

    其中commonLoader是其他两个的父加载器）

  * Thread.currentThread().setContextClassLoader(catalinaLoader);

    （设置线程上下文加载器,线程上下文加载器的设置主要是为了破坏双亲委派模型）

  * SecurityClassLoad.securityClassLoad(catalinaLoader);

    （防止加载需要的类被安全检测阻止）

  * 加载Catalina实例

* setAwait(true);（通过反射调用catalina实例的方法）

* load(args)（通过反射调用catalina实例的方法）

* start()（通过反射调用catalina实例的方法）

## catlina的执行

* setAwait(true);

  （将一个标识await 设置成true）

* load();

  （new Digester利用xml来初始化Server,以及Server下多个Service

  调用了getServer.init()->LifecycleBase->StandardServer->Server.inintInternal()->service.init()[多个]）

* start();

  （Server.start()->LifecycleBase->StandardServer->Server.startInternal()->service.start()[多个]）

  * 调用过程中不断的调用service的start()

## server的执行

* init()

  调用了getServer.init()->LifecycleBase->StandardServer->Server.inintInternal()->service.init()[多个]）

  ```java
  // Initialize our defined Services
  for (Service service : services) {
      service.init();
  }
  ```

* start();

  （Server.start()->LifecycleBase->StandardServer->Server.startInternal()->service.start()[多个]）

  * 调用过程中不断的调用service的start()

    ```java
    synchronized (servicesLock) {
        for (Service service : services) {
            service.start();
        }
    }
    ```

## service的执行

* init()->StandardService.initInternal();

  ```java
  protected void initInternal() throws LifecycleException {
      super.initInternal();
      
      if (engine != null) {
          engine.init();
      }
      // Initialize any Executors
      for (Executor executor : findExecutors()) {
          if (executor instanceof JmxEnabled) {
              ((JmxEnabled) executor).setDomain(getDomain());
          }
          executor.init();
      }
  
      // Initialize mapper listener
      mapperListener.init();
  
      // Initialize our defined Connectors
      synchronized (connectorsLock) {
          for (Connector connector : connectors) {
              try {
                  connector.init();
              } catch (Exception e) {
                  String message = sm.getString(
                      "standardService.connector.initFailed", connector);
                  log.error(message, e);
  
                  if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"))
                      throw new LifecycleException(message);
              }
          }
      }
  }
  ```

  * engine.init();**注：tomcat7中是直接container.init();**
    * 这里会调用super.initInternal()，也就是Container.initInternal()
  * 初始化Executor,创建线程池来管理connnector。
  * mapperListener.init()，监听container的变化
  * 初始化connnector（多个）

* start()->StandardService.startInternal()

  * 代码略过
  * engine.start()
  * executor.start()
  * mapperListener.start()
  * connector.start();


## Lifecycle

```
*            start()
*  -----------------------------
*  |                           |
*  | init()                    |
* NEW -»-- INITIALIZING        |
* | |           |              |     ------------------«-----------------------
* | |           |auto          |     |                                        |
* | |          \|/    start() \|/   \|/     auto          auto         stop() |
* | |      INITIALIZED --»-- STARTING_PREP --»- STARTING --»- STARTED --»---  |
* | |         |                                                            |  |
* | |destroy()|                                                            |  |
* | --»-----«--    ------------------------«--------------------------------  ^
* |     |          |                                                          |
* |     |         \|/          auto                 auto              start() |
* |     |     STOPPING_PREP ----»---- STOPPING ------»----- STOPPED -----»-----
* |    \|/                               ^                     |  ^
* |     |               stop()           |                     |  |
* |     |       --------------------------                     |  |
* |     |       |                                              |  |
* |     |       |    destroy()                       destroy() |  |
* |     |    FAILED ----»------ DESTROYING ---«-----------------  |
* |     |                        ^     |                          |
* |     |     destroy()          |     |auto                      |
* |     --------»-----------------    \|/                         |
* |                                 DESTROYED                     |
* |                                                               |
* |                            stop()                             |
* ----»-----------------------------»------------------------------
```

* 定义了13个常量，用于LifecycleEvent事件中type属性，

  好处：只需要发送一种类型事件LifecycleEvent,就可以判断是什么事件，否则每种事件都得需要定义类

* 定义了4种状态

  init(),start(),stop(),destroy()

* 定义了3种监听器相关的方法

  addLifecycleListener(),findLifecycleListeners(),removeLifecycleListener()

* 2种获取当前状态

  getState(),getStateName()

## LifecycleBase

* 实现Lifecycle接口的一个抽象类

* 默认实现了3个监听器方法的实现

  * 利用一个CopyOnWriteArrayList来保存监听器
  * 添加了fireLifecycleEvent()方法，处理lifecycleEvent时间的方法。
  
* **注：tomcat7中由LifecycleSupport来定义关于生命周期的方法**

* **注：tomcat7中存在LifecycleSupport，tomcat8.5中不存在LifecycleSupport**
  
* 4个生命周期方法：

  * init(),看一波源码

    ```java
    @Override
    public final synchronized void init() throws LifecycleException {
        if (!state.equals(LifecycleState.NEW)) {
            invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
        }
    
        try {
            setStateInternal(LifecycleState.INITIALIZING, null, false);
            initInternal();
            setStateInternal(LifecycleState.INITIALIZED, null, false);
        } catch (Throwable t) {
            handleSubClassException(t, "lifecycleBase.initFail", toString());
        }
    }
    ```

    * 将状态设置成了INITIALIZING
    
    * 然后执行具体的initInternal()
    
      **这里的initInternal都是指子类重写的方法**
    
    * 然后再将状态设置成INITIALIZED,初始化结束
    
  * start();看一波源码

    ```java
    @Override
    public final synchronized void start() throws LifecycleException {
        if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
            LifecycleState.STARTED.equals(state)) {
            if (log.isDebugEnabled()) {
                Exception e = new LifecycleException();
                log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
            } else if (log.isInfoEnabled()) {
                log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
            }
            return;
        }
        if (state.equals(LifecycleState.NEW)) {
            init();
        } else if (state.equals(LifecycleState.FAILED)) {
            stop();
        } else if (!state.equals(LifecycleState.INITIALIZED) &&
                   !state.equals(LifecycleState.STOPPED)) {
            invalidTransition(Lifecycle.BEFORE_START_EVENT);
        }
    
        try {
            setStateInternal(LifecycleState.STARTING_PREP, null, false);
            startInternal();
            if (state.equals(LifecycleState.FAILED)) {
                // This is a 'controlled' failure. The component put itself into the
                // FAILED state so call stop() to complete the clean-up.
                stop();
            } else if (!state.equals(LifecycleState.STARTING)) {
                // Shouldn't be necessary but acts as a check that sub-classes are
                // doing what they are supposed to.
                invalidTransition(Lifecycle.AFTER_START_EVENT);
            } else {
                setStateInternal(LifecycleState.STARTED, null, false);
            }
        } catch (Throwable t) {
            // This is an 'uncontrolled' failure so put the component into the
            // FAILED state and throw an exception.
            handleSubClassException(t, "lifecycleBase.startFail", toString());
        }
    }
    ```

    * 如果当前状态是start中的一种，就return

    * 然后根据不同的状态来执行不同的操作

    * 将状态设置为STARTING_PREP

    * startInternal();

      **这里的startInternal都是指子类重写的方法**

    * 根据不同的结果，执行不同的操作，如果正确执行，就将状态设置为STARTED

  * stop()

    ```java
     @Override
    public final synchronized void stop() throws LifecycleException {
        if (LifecycleState.STOPPING_PREP.equals(state) || LifecycleState.STOPPING.equals(state) ||
            LifecycleState.STOPPED.equals(state)) {
            if (log.isDebugEnabled()) {
                Exception e = new LifecycleException();
                log.debug(sm.getString("lifecycleBase.alreadyStopped", toString()), e);
            } else if (log.isInfoEnabled()) {
                log.info(sm.getString("lifecycleBase.alreadyStopped", toString()));
            }
            return;
        }
        if (state.equals(LifecycleState.NEW)) {
            state = LifecycleState.STOPPED;
            return;
        }
        if (!state.equals(LifecycleState.STARTED) && !state.equals(LifecycleState.FAILED)) {
            invalidTransition(Lifecycle.BEFORE_STOP_EVENT);
        }
        try {
            if (state.equals(LifecycleState.FAILED)) {
                // Don't transition to STOPPING_PREP as that would briefly mark the
                // component as available but do ensure the BEFORE_STOP_EVENT is
                // fired
                fireLifecycleEvent(BEFORE_STOP_EVENT, null);
            } else {
                setStateInternal(LifecycleState.STOPPING_PREP, null, false);
            }
            stopInternal();
            // Shouldn't be necessary but acts as a check that sub-classes are
            // doing what they are supposed to.
            if (!state.equals(LifecycleState.STOPPING) && !state.equals(LifecycleState.FAILED)) {
                invalidTransition(Lifecycle.AFTER_STOP_EVENT);
            }
            setStateInternal(LifecycleState.STOPPED, null, false);
        } catch (Throwable t) {
            handleSubClassException(t, "lifecycleBase.stopFail", toString());
        } finally {
            if (this instanceof Lifecycle.SingleUse) {
                // Complete stop process first
                setStateInternal(LifecycleState.STOPPED, null, false);
                destroy();
            }
        }
    }
    ```

    

  * destroy()

    ```java
    public final synchronized void destroy() throws LifecycleException {
        if (LifecycleState.FAILED.equals(state)) {
            try {
                // Triggers clean-up
                stop();
            } catch (LifecycleException e) {
                // Just log. Still want to destroy.
                log.error(sm.getString("lifecycleBase.destroyStopFail", toString()), e);
            }
        }
        if (LifecycleState.DESTROYING.equals(state) || LifecycleState.DESTROYED.equals(state)) {
            if (log.isDebugEnabled()) {
                Exception e = new LifecycleException();
                log.debug(sm.getString("lifecycleBase.alreadyDestroyed", toString()), e);
            } else if (log.isInfoEnabled() && !(this instanceof Lifecycle.SingleUse)) {
                // Rather than have every component that might need to call
                // destroy() check for SingleUse, don't log an info message if
                // multiple calls are made to destroy()
                log.info(sm.getString("lifecycleBase.alreadyDestroyed", toString()));
            }
            return;
        }
    
        if (!state.equals(LifecycleState.STOPPED) && !state.equals(LifecycleState.FAILED) &&
            !state.equals(LifecycleState.NEW) && !state.equals(LifecycleState.INITIALIZED)) {
            invalidTransition(Lifecycle.BEFORE_DESTROY_EVENT);
        }
    
        try {
            setStateInternal(LifecycleState.DESTROYING, null, false);
            destroyInternal();
            setStateInternal(LifecycleState.DESTROYED, null, false);
        } catch (Throwable t) {
            handleSubClassException(t, "lifecycleBase.destroyFail", toString());
        }
    }
    ```

## connector

Connector的主要功能，是接收连接请求，创建Request和Response对象用于和请求端交换数据；然后分配线程让Engine来处理这个请求，并把产生的Request和Response对象传给Engine。Engine是conntainer的子类，Container就是Servlet的容器，Container处理完后会把结果返回给Connector，最后Connector使用socket将处理结果返回给客户端，整个请求处理完毕。

Connector的底层是使用socket进行连接，Request和Response时按照http协议（协议可选）来封装，所以Connector同时实现了TCP/IP和HTTP协议。**(注：server.xml可以配置)**

```xml
 <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

* 最开始分析一波init();看一波源码

  ```java
  @Override
  protected void initInternal() throws LifecycleException {
  	//这里对父类进行初始化
      super.initInternal();
      // Initialize adapter
      adapter = new CoyoteAdapter(this);
      protocolHandler.setAdapter(adapter);
  
      // Make sure parseBodyMethodsSet has a default
      if (null == parseBodyMethodsSet) {
          setParseBodyMethods(getParseBodyMethods());
      }
  
      if (protocolHandler.isAprRequired() && !AprLifecycleListener.isInstanceCreated()) {
          throw new LifecycleException(sm.getString("coyoteConnector.protocolHandlerNoAprListener",
                  getProtocolHandlerClassName()));
      }
      if (protocolHandler.isAprRequired() && !AprLifecycleListener.isAprAvailable()) {
          throw new LifecycleException(sm.getString("coyoteConnector.protocolHandlerNoAprLibrary",
                  getProtocolHandlerClassName()));
      }
      if (AprLifecycleListener.isAprAvailable() && AprLifecycleListener.getUseOpenSSL() &&
              protocolHandler instanceof AbstractHttp11JsseProtocol) {
          AbstractHttp11JsseProtocol<?> jsseProtocolHandler =
                  (AbstractHttp11JsseProtocol<?>) protocolHandler;
          if (jsseProtocolHandler.isSSLEnabled() &&
                  jsseProtocolHandler.getSslImplementationName() == null) {
              // OpenSSL is compatible with the JSSE configuration, so use it if APR is available
      jsseProtocolHandler.setSslImplementationName(OpenSSLImplementation.class.getName());
          }
      }
  
      try {
          protocolHandler.init();
      } catch (Exception e) {
          throw new LifecycleException(
                  sm.getString("coyoteConnector.protocolHandlerInitializationFailed"), e);
      }
  }
  ```

  * 创建一个Adapter，并且设置到ProtocolHandler

  * 进行ProtocolHandler.init();

    * ProtocolHandler.init()->AbstractAjpProtocol.init()->AbstractHttp11Protocol.init()(当协议为http)

    * 在AbstractAjpProtocol.init()中会调用endpoint.init();

      ```java
    endpoint.setName(endpointName.substring(1, endpointName.length()-1));
      endpoint.setDomain(domain);
    
      endpoint.init();
      ```
    * endpoint.init()中会使用bind()方法(**这里bind()会调用其子类【3种】的bind()**)
    
      ![image-20210402185851676](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210402185851676.png)
    
      这是NioEndpoint中的一部分代码
    
      ```java
      // Initialize thread count defaults for acceptor, poller
      if (acceptorThreadCount == 0) {
          // FIXME: Doesn't seem to work that well with multiple accept threads
          acceptorThreadCount = 1;
      }
      if (pollerThreadCount <= 0) {
          //minimum one poller thread
          pollerThreadCount = 1;
      }
      ```
    
      **重要作用就是设置Acceptor和poller的默认数量/最小数量**
    
      * 注：accpetor的主要作用控制与tomcat建立连接的数量
      * 注：poller是负责socket的读写
    
  * 基本上connector的init()，就算结束了。
  
    **注：在tomcat7中，其实在connnector中在protocolHandler.init()之后还有mapperListener.init();**
  
    **但是在tomcat8.5中connector取消了mapperListener，所以也不存在mapperListener.init()**

* 分析一波start()

  ```java
  @Override
  protected void startInternal() throws LifecycleException {
      // Validate settings before starting
      if (getPort() < 0) {
          throw new LifecycleException(sm.getString(
                  "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
      }
      setState(LifecycleState.STARTING);
      try {
          protocolHandler.start();
      } catch (Exception e) {
          throw new LifecycleException(
                  sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
      }
  }
  ```

  **由上可知，其实connector.start()主要就是执行protocolHandler.start()**

  protocolHandler.start()->AbstractAjpProtocol.start()->AbstractProtocol.start()

  其中AbstractAjpProtocol是AbstractProtocol的子类，AbstractAjpProtocol会回调AbstractProtocol的start();

  看一下AbstractProtocol.start()的代码

  ```java
      @Override
      public void start() throws Exception {
          if (getLog().isInfoEnabled()) {
              getLog().info(sm.getString("abstractProtocolHandler.start", getName()));
          }
          endpoint.start();
          // Start timeout thread
          asyncTimeout = new AsyncTimeout();
          Thread timeoutThread = new Thread(asyncTimeout, getNameInternal() + "-AsyncTimeout");
          int priority = endpoint.getThreadPriority();
          if (priority < Thread.MIN_PRIORITY || priority > Thread.MAX_PRIORITY) {
              priority = Thread.NORM_PRIORITY;
          }
          timeoutThread.setPriority(priority);
          timeoutThread.setDaemon(true);
          timeoutThread.start();
      }
  ```

  可以看到在AbstractProtocol.start()中其实又调用了一次endpoint.start();

  ```java
  public final void start() throws Exception {
      if (bindState == BindState.UNBOUND) {
          bind();
          bindState = BindState.BOUND_ON_START;
      }
      startInternal();
  }
  ```

  执行startInternal();

  ![image-20210403101127394](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210403101127394.png)

  看一波NioEndpoint的startInternal源码

  ```java
  @Override
  public void startInternal() throws Exception {
      if (!running) {
          running = true;
          paused = false;
  
          processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                  socketProperties.getProcessorCache());
          eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                          socketProperties.getEventCache());
          nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                  socketProperties.getBufferPool());
  
          // Create worker collection
          if (getExecutor() == null) {
              createExecutor();
          }
          initializeConnectionLatch();
          // Start poller threads
          pollers = new Poller[getPollerThreadCount()];
          for (int i=0; i<pollers.length; i++) {
              pollers[i] = new Poller();
              Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
              pollerThread.setPriority(threadPriority);
              pollerThread.setDaemon(true);
              pollerThread.start();
          }
  
          startAcceptorThreads();
      }
  }
  ```

  其实可以分析出来，其NioEndpoint.startInternal主要的操作有

  1，初始化一些属性

  2，创建一些缓冲池**(注：tomcat7中没有这个部分)**

  ```java
  processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                                       socketProperties.getProcessorCache());
  eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                                       socketProperties.getEventCache());
  nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                                        socketProperties.getBufferPool());
  ```

  3，创建线程池

  4，根据之前定义的acceptor和poller的线程数量，来启动相应的acceptor和poller。

* 到这里connector的初始化基本完成

  

  这里有对connector，poller解释比较详细的博客

  https://cloud.tencent.com/developer/inventory/1620/article/1685912

  https://cloud.tencent.com/developer/article/1690229

## container

### container的结构

container是一个接口，所有子容器都继承这个接口。并且container这个接口继承了Lifecycle接口，所以container，以及它的子容器 也具有生命周期。

container有一个默认的实现类，ContainerBase

子容器有4种 Engine，Host，Context，Wrapper。

每个service只能有一个Engine，而一个Engine可以有多个Host，一个Host可以有多个Context，每个Context能够含有多个Wrapper

![image-20210403173325277](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210403173325277.png)

* Engine表示引擎，可以来管理多个host（虚拟主机），用来接收connet传来的request和response，选择可用的host还处理请求

* 每个host可以部署多个context（也就是多个应用程序）

  比如一个www.a.com就是表示一个host，而www.a.com/login就是一个应用

* 一个context可以有多个wrapper

* 每个wrapper就封装着一个servlet

### 关于几个容器的生命周期

这里讲一下关于几个容器的生命周期

1，虽然子容器都有相同的父类ContainerBase定义共同的initInternal和startInternal()等方法，但是每个容器其实还有自定义添加了其他的内容

2，关于初始化方面，其实上文说过，我们在service初始化的时候会调用Engine的初始化，但是在进行这种父容器的初始化的时候是不会一起初始化子容器的。

以下是Engine的初始化，可以发现其实没有对子容器的任何操作。

```java
    @Override
    protected void initInternal() throws LifecycleException {
        // Ensure that a Realm is present before any attempt is made to start
        // one. This will create the default NullRealm if necessary.
        getRealm();
        super.initInternal();
    }
```

### 关于container的初始化

方式1：在Service.init()->Engine.init()->Contain.init() **(注意，这里由子类调用父类的初始化)**

然后再Service.start()->Engine.start()->Contain.start()->子容器的初始化

```java
// Start our child containers, if any
Container children[] = findChildren();
List<Future<Void>> results = new ArrayList<>();
for (Container child : children) {
    results.add(startStopExecutor.submit(new StartChild(child)));
}
```

**以上代码截取自ContainerBase.startInternal();关于子容器的初始化**

由此就可以完成容器的初始化。

这里只是讲一下关于容器初始化的流程，对其中的细节并未表述。

来尝试分析一波。

* Container.init();

  * 执行流程：Service.init()-> Engine.init()->Container.init()->ContainerBase.initInternal();

    * 以下就是ContainerBase.initInternal()

    ```java
        @Override
        protected void initInternal() throws LifecycleException {
            BlockingQueue<Runnable> startStopQueue = new LinkedBlockingQueue<>();
            startStopExecutor = new ThreadPoolExecutor(
                    getStartStopThreadsInternal(),
                    getStartStopThreadsInternal(), 10, TimeUnit.SECONDS,
                    startStopQueue,
                    new StartStopThreadFactory(getName() + "-startStop-"));
            startStopExecutor.allowCoreThreadTimeOut(true);
            super.initInternal();
        }
    ```

    这里很明显是创建了一个线程池用来管理启动和关闭线程

* Container.start();

  * 执行流程Service.start()->Engine.start()->Container.start()->ContainerBase.startInternal();

    **代码如下：**

    ```java
        @Override
        protected synchronized void startInternal() throws LifecycleException {
    
            // 如果存在Cluster，Realm，就启动
            logger = null;
            getLogger();
            Cluster cluster = getClusterInternal();
            if (cluster instanceof Lifecycle) {
                ((Lifecycle) cluster).start();
            }
            Realm realm = getRealmInternal();
            if (realm instanceof Lifecycle) {
                ((Lifecycle) realm).start();
            }
    
            // 利用线程池启动子容器
            Container children[] = findChildren();
            List<Future<Void>> results = new ArrayList<>();
            for (Container child : children) {
                results.add(startStopExecutor.submit(new StartChild(child)));
            }
    
            MultiThrowable multiThrowable = null;
    
            for (Future<Void> result : results) {
                try {
                    result.get();
                } catch (Throwable e) {
                    log.error(sm.getString("containerBase.threadedStartFailed"), e);
                    if (multiThrowable == null) {
                        multiThrowable = new MultiThrowable();
                    }
                    multiThrowable.add(e);
                }
    
            }
            if (multiThrowable != null) {
                throw new LifecycleException(sm.getString("containerBase.threadedStartFailed"),
                        multiThrowable.getThrowable());
            }
    
            // 调用管道
            if (pipeline instanceof Lifecycle) {
                ((Lifecycle) pipeline).start();
            }
    
    		//设置状态
            setState(LifecycleState.STARTING);
    
            // 启动线程
            threadStart();
        }
    
    ```

    * 如果存在Cluster和Realm，就启动他们

      Cluster用于配置集群，在server.xml中可配置，它的作用是同步session。

      Realm是tomcat的安全域，可以用来管理资源的访问权限。

    * 使用startStopExecutor调用新的线程来启动每一个子容器（也就是调用他们的start()方法）

    * 启动管道

    * 设置生命周期状态为STARTING

    * 调用threadStart的方法启动后台线程，循环调用一个backgroundProcess(),

      **（为了篇幅逻辑流畅，不在此处做表述，详细查看container中threadStart这一节）**

  到这里基本上已经讲完了Container的启动。

  接下来分析关于Container的子容器
  
* StandardEngine

  * init();

    ```java
    @Override
    protected void initInternal() throws LifecycleException {
        // Ensure that a Realm is present before any attempt is made to start
        // one. This will create the default NullRealm if necessary.
        getRealm();
        super.initInternal();
    }
    ```

    在进行初始化时候先调用了getRealm()方法；作用就是如果没有配置Realm，就会创建一个默认的NullPealm。然后调用了父类的初始化方法（也就是Container的initInternal()）。

  * start();

    ```java
    @Override
    protected synchronized void startInternal() throws LifecycleException {
        // Log our server identification information
        if(log.isInfoEnabled())
            log.info( "Starting Servlet Engine: " + ServerInfo.getServerInfo());
        // Standard container startup
        super.startInternal();
    }
    ```

    这里可以看出来，其实这里只是调用了父类的startInternal()方法。

* StandardHost

  * init()

  ![image-20210404204209820](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210404204209820.png)

  ​	可以发现StandardHost没有重写initInternal()方法，所以这里调用的是父类的initInternal()方法，也就是ContainerBase的initInternal()方法。

  * start();

    ```java
    @Override
    protected synchronized void startInternal() throws LifecycleException {
    
        // Set error report valve
        String errorValve = getErrorReportValveClass();
        if ((errorValve != null) && (!errorValve.equals(""))) {
            try {
                //判断管道中是否存在对应的errorValve
                boolean found = false;
                Valve[] valves = getPipeline().getValves();
                for (Valve valve : valves) {
                    if (errorValve.equals(valve.getClass().getName())) {
                        found = true;
                        break;
                    }
                }
                //如果不存在就添加
                if(!found) {
                    Valve valve =
                        (Valve) Class.forName(errorValve).getConstructor().newInstance();
                    getPipeline().addValve(valve);
                }
            } catch (Throwable t) {
                ExceptionUtils.handleThrowable(t);
                log.error(sm.getString(
                        "standardHost.invalidErrorReportValveClass",
                        errorValve), t);
            }
        }
        
        super.startInternal();
    }
    ```
    
    其中关键的代码就是try中的那一部分，主要的功能就是通过foreach循环检查**管道**中是否存在**errorValve**，如果没有找到，就添加对应的valve到管道里面。

* StandardContext

  * 同样这里面也没有重写initInternal()
  * startInternal（）

  ![image-20210413102526935](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210413102526935.png)

  ​	可以看出来这是一段非常长的代码。所以这里就不全贴出来代码了。

  ​	简单分析一下

  ​	1,前面进行了一些验证保证加载资源的操作，这里不做太多描述

  ​	2,postWorkDirectory();为我们的工作目录设置适当的上下文属性。	

  ​	流程：

  ​	（1）获得工作目录	

  ​	（2）如果有必要的话会创建这个目录

  ​	（3）设置servletContext()

  ​	3,添加缺少的组件和资源

  ​	4,设置加载器

  ​	5,做一些初始化工作，cookieProcessor的设置，字符串集合映射，扩展验证等

  ​	6,将Thread和线程上下文加载器进行一个捆绑。

  ​	7,将Thread设置为webapp加载器

  ​	8,通知对应的LifecycleListeners（监听器）

  ​	9,启动未启动的子容器

  ​	10,启动管道

  ​	11,获得cluster的管理器

  ​	12,将资源添加到servlet的context中

  ​    13,设置相关的context参数

  ​    14,调用servlet容器的初始化器

  ​	15,配置并且调用application 事件的监听器

  ​	16,查看是否有编程上的对于http的约束条件 

  ​	17,启动cluster管理器

  ​	18,配置和启动application的过滤器

  ​	19,启动那些在启动就需要加载的servlet

  ​	20,设置启动成功的状态标识

  ​	21,对资源进行一个gc操作

  ​	22,如果有问题，就重新初始化

  ​	**只能对流程的一个简单分析，至于其中的具体实现，此处就不做展开**

  可以看出来其实在其中的操作有很多，但是其中的核心其实就是初始化Servlet（调用那些需要在启动时就加载的servlet）。

* StandardWrapper

  * 同样对于initInternal()是没有重写的。

  * startInternal()

    ```java
    protected synchronized void startInternal() throws LifecycleException {
    
            // Send j2ee.state.starting notification
            if (this.getObjectName() != null) {
                Notification notification = new Notification("j2ee.state.starting",
                                                            this.getObjectName(),
                                                            sequenceNumber++);
                broadcaster.sendNotification(notification);
            }
    
            // Start up this component
            super.startInternal();
    
            setAvailable(0L);
    
            // Send j2ee.state.running notification
            if (this.getObjectName() != null) {
                Notification notification =
                    new Notification("j2ee.state.running", this.getObjectName(),
                                    sequenceNumber++);
                broadcaster.sendNotification(notification);
            }
    
        }
    ```

    可以看出来这里面的操作还是比较简单的。

    * 利用broadcaser进行一个通知
    * 调用父类的startInternal()
    * 调用setAvailable(),作用是设置Wrapper所包含的所有servlet的有效其实时间。





### container中threadStart详细介绍

* 关于threadStart()作用的位置，这里不做介绍，在上文container.start()中有详细介绍

* 以下是ContainerBackgroundProcessor的核心代码

* ```java
  while (!threadDone) {
      try {
          Thread.sleep(backgroundProcessorDelay * 1000L);
      } catch (InterruptedException e) {
          // Ignore
      }
      if (!threadDone) {
          processChildren(ContainerBase.this);
      }
  }
  ```

* 可以看出该线程是一个while循环，定期调用processChildren方法做一些事情，间隔时间可通过属性设置，单位是秒，如果小于0就不启动后台线程了。

* processChildren()的核心部分其实是container.backgroundProcess();

* 也就是循环的调用backgroundProcess();

* 而backgroundProcess在ContainerBase、StandardContext、StandardWrapper中都有实现。

  ![image-20210404110505056](https://gitee.com/stiwen/images_bed/raw/master/img/image-20210404110505056.png)
  **在ContainerBase中backgroundProcess中**

  ```java
  @Override
  public void backgroundProcess() {
  
      if (!getState().isAvailable())
          return;
  
      Cluster cluster = getClusterInternal();
      if (cluster != null) {
          try {
              cluster.backgroundProcess();
          } catch (Exception e) {
              log.warn(sm.getString("containerBase.backgroundProcess.cluster",
                      cluster), e);
          }
      }
      Realm realm = getRealmInternal();
      if (realm != null) {
          try {
              realm.backgroundProcess();
          } catch (Exception e) {
              log.warn(sm.getString("containerBase.backgroundProcess.realm", realm), e);
          }
      }
      Valve current = pipeline.getFirst();
      while (current != null) {
          try {
              current.backgroundProcess();
          } catch (Exception e) {
              log.warn(sm.getString("containerBase.backgroundProcess.valve", current), e);
          }
          current = current.getNext();
      }
      fireLifecycleEvent(Lifecycle.PERIODIC_EVENT, null);
  }
  ```
  * 调用了**Cluster**和**Realm**和**管道**的backgroundProcess()方法

  **在StandardContext中的backgroundProcess**

  * 调用了loader.backgroundProcess();

    * 设置线程上下文加载器，并且不断加载

  * manager.backgroundProcess();

    * 根据设定的一个频率不断的查看，是过期的sessions无效

  * resources.backgroundProcess(）;

    * 会调用cache.backgroundProcess();

      **收集最近最少使用的资源**

    * gc();

      **垃圾收集**

  * ((DefaultInstanceManager)instanceManager).backgroundProcess();

    * 从虚引用中，清除一些dead keys

  **在StandardWrapper中的backgroundProcess**

  ```java
    @Override
      public void backgroundProcess() {
          super.backgroundProcess();
  
          if (!getState().isAvailable())
              return;
  
          if (getServlet() instanceof PeriodicEventListener) {
              ((PeriodicEventListener) getServlet()).periodicEvent();
          }
      }
  
  
      @Override
      public void periodicEvent() {
          //后台线程用于检查是否应卸载任何JSP的方法
          rctxt.checkUnload();
          //检测注册的jsp的依赖
          rctxt.checkCompile();
      }
  ```

  * 其实就是对jsp生成的seervlet定期检查。

  **由此对于后台线程的执行介绍完毕**

参考来源：

https://cloud.tencent.com/developer/article/1448339

https://time.geekbang.org/column/article/96764

