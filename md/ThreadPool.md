# 使用线程池导致的获取用户信息异常

 - [应用场景](#应用场景)
 
 - [线上出现问题](#线上出现问题)
  
 - [解决方法](#解决方法)
 
 ### 应用场景
     由于针对IDP批量发送消息，存在消息量比较大，此时就需要对IDP消息进行线程池异步化处理
 
 ### 线上出现问题
     由于用到线程池但是不同的线程在处理用户信息的时候就会出现用户的权限问题，导致线上发送消息异常 
     
     192.168.X.X 2020-04-28 18:03:34:648 ERROR [PASSPORT_LIEPINHandler-16] (BoleLoggerBizImpl.java:83)
      - 没有操作权限 没有操作权限
     bs.1588068211251.2.1 at com.xx.prophet.platform.support.idp.PASSPORT_XXHandler.lambda$handleUSER_LOGIN$4
     (PASSPORT_LIEPINHandler.java:98)
      com.xx.swift.core.exception.BizException: 没有操作权限
              at com.xx.swift.framework.rpc.proxy.ServiceInterceptor$HystrixCommandCallWrapper.run
              (ServiceInterceptor.java:249)
              at com.netflix.hystrix.HystrixCommand$1.call(HystrixCommand.java:293)
              at com.netflix.hystrix.HystrixCommand$1.call(HystrixCommand.java:288)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at com.netflix.hystrix.AbstractCommand$1.call(AbstractCommand.java:388)
              at com.netflix.hystrix.AbstractCommand$1.call(AbstractCommand.java:369)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41)
              at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30)
              at rx.Observable.unsafeSubscribe(Observable.java:10327)
              at com.netflix.hystrix.AbstractCommand$ObservableCommand$1.call(AbstractCommand.java:1133)
              at com.netflix.hystrix.AbstractCommand$ObservableCommand$1.call(AbstractCommand.java:1129)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48)
              at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
              at rx.Observable.subscribe(Observable.java:10423)
              at rx.Observable.subscribe(Observable.java:10390)
              at rx.internal.operators.BlockingOperatorToFuture.toFuture(BlockingOperatorToFuture.java:51)
              at rx.observables.BlockingObservable.toFuture(BlockingObservable.java:410)
              at com.netflix.hystrix.HystrixCommand.queue(HystrixCommand.java:378)
              at com.netflix.hystrix.HystrixCommand.execute(HystrixCommand.java:334)
              at com.xx.swift.framework.rpc.proxy.ServiceInterceptor.invoke(ServiceInterceptor.java:89)
              at com.sun.proxy.$Proxy149.findDefaultResIdsByUserIds(Unknown Source)
              at com.xx.prophet.platform.biz.impl.ResumeHandlerBizImpl.commonHandlerUserIdsCache
              (ResumeHandlerBizImpl.java:95)
              at com.xx.prophet.platform.biz.impl.ResumeHandlerBizImpl.commonHandlerUserIds
              (ResumeHandlerBizImpl.java:80)
              at com.xx.prophet.platform.biz.impl.ResumeHandlerBizImpl$$FastClassBySpringCGLIB$$a222d23c.
              invoke(<generated>)
              at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
              at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint
              (CglibAopProxy.java:749)
              at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed
              (ReflectiveMethodInvocation.java:163)
              at com.xx.swift.framework.monitor.cat.CatAdvice.invoke(CatAdvice.java:28)
              at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed
              (ReflectiveMethodInvocation.java:186)
              at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.
              intercept(CglibAopProxy.java:688)
              at com.xx.prophet.platform.biz.impl.ResumeHandlerBizImpl$$EnhancerBySpringCGLIB$$73221940.
              commonHandlerUserIds(<generated>)
              at com.xx.prophet.platform.support.idp.PASSPORT_LIEPINHandler.lambda$handleUSER_LOGIN$4
              (PASSPORT_LIEPINHandler.java:96)
              at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
              at java.util.concurrent.FutureTask.run(FutureTask.java:266)
              at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
              at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
              at java.lang.Thread.run(Thread.java:748)
  
 ### 解决方法

        @Service("IDPHandler.XX.*")
        public class ICVIDPHandler extends MultiIDPHandler {
        
            private static final MonitorLogger MONITOR = MonitorLogger.getInstance();
            private static ListeningExecutorService THREAD_POOL = MoreExecutors.listeningDecorator(
                    new ThreadPoolExecutor(20,
                            20,
                            30,
                            TimeUnit.MILLISECONDS,
                            new LinkedBlockingDeque<>(100),
                            new NamedThreadFactory("IXXIDPHandler"),
                            (r, executor) -> MONITOR.log("WARN 警告 " + executor + "线程池负载过高，
                            任务被拒绝：" + r.toString())
                    ));
        
            /**
             * <b>UPDATE_INDEX批量</b>
             */
            public void handleUPDATE_INDEX(List<IdpMessage> idpMessages) throws BizException {
                //在此处进行线程初始化获取用户信息，具体ThreadLocalUtil代码不做展示
                Map<String, Object> tl = ThreadLocalUtil.getInstance().getAll();
                THREAD_POOL.submit(() -> {
                    try {
                        //每个线程调用之前set处理
                        ThreadLocalUtil.getInstance().set(tl);
                        resumeHandlerBiz.commonHandlerResIds(idpMessages);
                    } catch (BizException e) {
                        boleLoggerBiz.error(e, e.getMessage());
                    }
                });
            }
        
        }

       
> reubenwang@foxmail.com
> 没事别找我，找我也不在！--我很忙🦆