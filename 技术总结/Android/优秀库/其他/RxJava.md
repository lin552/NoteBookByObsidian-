---
创建时间: 2025-05-09 17:39:24
作者: wangxiaoming
tags:
  - RxJava
---
#### 一、`RxJava`概述
- ​**定义**​：基于响应式编程的库，通过观察者模式处理异步数据流，简化事件驱动和异步编程 
- ​**核心思想**​：将数据流视为可观察的序列，通过订阅（Subscribe）建立观察者（Observer）与被观察者（Observable）的连接，实现事件传递 
#### 二、核心概念及详情

| ​**概念**​           | ​**作用**​                                                                 | ​**关键方法/类型**​                                     |
| ------------------ | ------------------------------------------------------------------------ | ------------------------------------------------- |
| ​**Observable**​   | 数据生产者，发射事件（Next/Error/Complete）                                          | `Observable.create()`、`just()`、`fromIterable()`   |
| ​**Observer**​     | 数据消费者，处理事件                                                               | 实现 `onNext()`、`onError()`、`onComplete()`          |
| ​**Subscription**​ | 管理订阅关系，支持取消订阅                                                            | `dispose()` 释放资源                                  |
| ​**Scheduler**​    | 控制线程切换（如 `Schedulers.io()` 处理IO，`AndroidSchedulers.mainThread()` 更新`UI`） | `subscribeOn()` 指定执行线程，`observeOn()` 指定观察线程       |
| ​**操作符**​          | 转换、过滤、合并数据流                                                              | `map()`（类型转换）、`filter()`（筛选）、`flatMap()`（扁平化嵌套数据） |
##### 1）Observable 数据生产者
|型|特性|适用场景|数据量|错误处理|完成信号|
|---|---|---|---|---|---|
|​**Observable**​|多值流|实时数据流/事件流|0-N|支持|可选|
|​**Flowable**​|支持背压的多值流|高并发/数据洪峰场景|0-N|必须|可选|
|​**Single**​|单值或错误|确定性结果（如API响应）|0/1|支持|不支持|
|​**Maybe**​|0/1 值或错误|可能空结果查询|0/1|支持|可选|
|​**Completable**|仅完成/错误信号|无数据操作（如文件删除）|0|支持|必须|

| 数据特征      | 推荐方式                         | 典型场景案例                  |
| --------- | ---------------------------- | ----------------------- |
| 固定数据集     | `just()`/`fromIterable`      | 配置参数加载、静态列表展示           |
| 连续数字序列    | `range()`                    | 分页编号生成                  |
| 定时/周期任务   | `interval()`                 | 心跳检测、轮询接口               |
| 用户交互/系统事件 | `fromAction()`               | 按钮点击、键盘输入               |
| 完全自定义发射逻辑 | `create()`                   | 复杂状态机、协议解析              |
| 按需延迟创建    | `defer()`                    | 避免重复计算、动态参数             |
| 异步计算结果    | `fromFuture()`               | 线程池任务结果监听               |
| 类型转换      | `fromMaybe()`                | 将`Maybe`转换成`Observable` |
| 从其他类型转换   | `fromArray()/fromIterable()` | 将数组转换成数据流，或是其他          |
###### 代码使用示例
```java
//just() 发射固定数据项  
Observable.just(1,2,3,4,5)  
        .subscribe(System.out::println);  
//fromArray() 数组转换为数据流  
List<Integer> numbers = Arrays.asList(1,3,5,7,9);  
Observable.fromArray(numbers.toArray(new String[0])).subscribe(System.out::println);
//fromIterable() 发射集合元素  
List<Integer> numbers = Arrays.asList(1,3,5,7,9);  
Observable.fromIterable(numbers)  
        .filter(n->n % 2 == 1)  
        .subscribe(System.out::println);  
//range() 生成数字序列  
Observable.range(5,3)  
        .map(n-> "Item "+n)  
        .subscribe(System.out::println);  
//interval() 定时发射递增数字  
Observable.interval(1, TimeUnit.SECONDS)  
        .take(3) //仅接收前3个  
        .subscribe(tick -> System.out.println("Tick "+tick));  
//create() 自定义发射逻辑  
Observable.create(emitter -> {  
            try {  
                for (int i = 0; i < 5; i++) {  
                    if (emitter.isDisposed()) return;  
                    emitter.onNext(i *10);  
                }  
                emitter.onComplete();  
            } catch (Exception e) {  
                emitter.onError(e);  
            }  
        }  
        ).subscribe(  
                value -> System.out.println("Received:"+ value),  
        Throwable::printStackTrace,  
        ()-> System.out.println("Completed")  
);  
//formAction 来自某动作  
Observable.fromAction(() -> System.out.println("Button Clicked!")).subscribe(result -> System.out.println("Received: "+result),  
        Throwable::printStackTrace);  
//defer() 延迟创建Observable  
Observable.defer(() -> {  
    //避免重复计算，每次都是使用一开始生成的randomNum  
    int randomNum = new Random().nextInt(100);  
    return Observable.just("Random: "+randomNum);  
}).subscribe(System.out::println);  
//fromFuture() 转换Future结果  
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {  
    try {  
        Thread.sleep(1000);  
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    }  
    return "Future Result";  
});  
Observable.fromFuture(future)  
        .subscribe(System.out::println);  
//fromMaybe 将Maybe 转换为 ObservableMaybe<String> maybeData = Maybe.just("Legacy Data");  
Observable<String> maybeObservable = Observable.fromMaybe(maybeData);
```
##### 2）Observer 数据消费者
| Observer 类型        | 内存占用 | 启动时间 | 适合场景      |
| ------------------ | ---- | ---- | --------- |
| 基础 Observer        | 中    | 慢    | 需要完整控制    |
| DisposableObserver | 低    | 快    | 需要自动资源管理  |
| Subject            | 中    | 中    | 多订阅者共享数据流 |
| Lambda 表达式         | 最低   | 最快   | 简单数据流处理   |
###### ① Observer 代码使用示例
```java
//1.完整实现Observer  
Observer<String> observer = new Observer<String>() {  
    @Override  
    public void onSubscribe(@NonNull Disposable d) {  
        System.out.println("开始订阅");  
    }  
  
    @Override  
    public void onNext(@NonNull String s) {  
        System.out.println("接收数据：" + s);  
    }  
  
    @Override  
    public void onError(@NonNull Throwable e) {  
        System.out.println("发生错误：" + e);  
    }  
  
    @Override  
    public void onComplete() {  
        System.out.println("流完成");  
    }  
};  

//2.Lambda实现Observer  
Observable.just("A", "B")  
        .subscribe(  
                s -> System.out.println("接收：" + s),  
                e -> System.out.println("错误：" + e),  
                () -> System.out.println("完成")  
        );
        
//3.DisposableObserver 自动资源管理
DisposableObserver<Integer> observer = new DisposableObserver<Integer>() {  
    @Override  
    public void onNext(@NonNull Integer integer) {  
        System.out.println("处理值："+integer);  
    }  
  
    @Override  
    public void onError(@NonNull Throwable e) {  
        System.out.println("错误："+e);  
    }  
  
    @Override  
    public void onComplete() {  
        System.out.println("完成");  
    }  
};  
// 订阅时自动管理Disposable  
Observable observable = Observable.just(1);  
observable.subscribe(observer);

//4.Suject 同时具备 Observer 和 Observable 能力，支持多播
PublishSubject<String> subject = PublishSubject.create();  
//作为 Observer 订阅  
subject.subscribe(s -> System.out.println("收到："+s));  
//作为 Observable 发射数据  
subject.onNext("Hello");  
subject.onNext("Rxjava");
```
##### 3）Subscription 管理订阅关系
###### ① 代码使用示例
```java
//1.Rxjava 2.x 用法
//创建 
ObservableObservable<String> observable = Observable.just("Hello Rxjava");  
//订阅获取 
DisposableDisposable disposable = observable  
        .subscribeOn(Schedulers.io())  
        .observeOn(AndroidSchedulers.mainThread())  
        .subscribe(  
                System.out::println,  
                Throwable::printStackTrace  
        );  
//取消订阅  
disposable.dispose();

//2.合并多个
Observable<String> observable1 = Observable.just("A");  
Observable<String> observable2 = Observable.just("B");  
Disposable disposable1 = observable1  
        .subscribeOn(Schedulers.io())  
        .observeOn(AndroidSchedulers.mainThread())  
        .subscribe(  
                System.out::println,  
                Throwable::printStackTrace  
        );  
Disposable disposable2 = observable2  
        .subscribeOn(Schedulers.io())  
        .observeOn(AndroidSchedulers.mainThread())  
        .subscribe(  
                System.out::println,  
                Throwable::printStackTrace  
        );   
CompositeDisposable composite = new CompositeDisposable();  
//添加订阅  
composite.add(disposable1);  
composite.add(disposable2);  
//批量取消  
composite.clear();

//3.使用其他库，自动管理
// 使用 AutoDispose 库（需添加依赖）
observable
    .compose(AutoDispose.autoDisposable(AndroidLifecycleScopeProvider.from(this)))
    .subscribe(...);
```
##### 4） 操作符
详情查工具书，没必要一个一个写了
#### 三、`RxJava`核心特性
1. ​**异步编程**​
    - 避免回调地狱，通过链式调用（如 `map().filter().subscribe()`）清晰表达逻辑 
2. ​**操作符生态**​
    - 提供 ​**80+ 操作符**，支持复杂数据流处理（如 `zip()` 合并多流、`debounce()` 防抖）
3. ​**线程控制**​
    - 通过调度器实现线程切换，例如：
```java
   Observable.just("Hello")
     .subscribeOn(Schedulers.io())   // IO线程执行
     .observeOn(AndroidSchedulers.mainThread())  // 主线程更新UI
      .subscribe();
```
4. ​**背压机制（`Flowable`）​**​
    - 处理高并发场景，通过 `BackpressureStrategy` 控制数据生产与消费速率，避免内存溢出 
5. ​**错误处理**​
    - 链式传递错误，支持 `retry()` 重试、`onErrorReturn()` 返回默认值 

