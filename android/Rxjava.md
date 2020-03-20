# Rxjava
项目地址：https://github.com/ReactiveX/RxAndroid

- [Rxjava](#rxjava)
  - [定义](#%e5%ae%9a%e4%b9%89)
  - [基本概念](#%e5%9f%ba%e6%9c%ac%e6%a6%82%e5%bf%b5)
    - [四个基本概念](#%e5%9b%9b%e4%b8%aa%e5%9f%ba%e6%9c%ac%e6%a6%82%e5%bf%b5)
    - [事件回调方法](#%e4%ba%8b%e4%bb%b6%e5%9b%9e%e8%b0%83%e6%96%b9%e6%b3%95)
  - [基本实现](#%e5%9f%ba%e6%9c%ac%e5%ae%9e%e7%8e%b0)
    - [创建观察者（Observer/Subscriber）](#%e5%88%9b%e5%bb%ba%e8%a7%82%e5%af%9f%e8%80%85observersubscriber)
    - [创建具体目标（Observable）](#%e5%88%9b%e5%bb%ba%e5%85%b7%e4%bd%93%e7%9b%ae%e6%a0%87observable)
    - [创建订阅（Subscribe）](#%e5%88%9b%e5%bb%ba%e8%ae%a2%e9%98%85subscribe)
    - [简单的同步调用应用举例](#%e7%ae%80%e5%8d%95%e7%9a%84%e5%90%8c%e6%ad%a5%e8%b0%83%e7%94%a8%e5%ba%94%e7%94%a8%e4%b8%be%e4%be%8b)
  - [异步调用](#%e5%bc%82%e6%ad%a5%e8%b0%83%e7%94%a8)
    - [线程控制-Scheduler](#%e7%ba%bf%e7%a8%8b%e6%8e%a7%e5%88%b6-scheduler)

## 定义

>Rxjava：一个基于观察者模式，用来实现异步操作的库

## 基本概念 
### 四个基本概念
Rxjava有四个基本概念，分别是Observable (具体目标，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。其中observable和observer通过subscribe()实现订阅关系, Observable 可以在需要的时候发出事件来通知 Observer。<br>

### 事件回调方法
 RxJava 的事件回调方有三个：<br>
 - **onNext()**:普通事件。相当于Onclick()/onEvent()<br>
 - **onCompleted()**:事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志。<br> 
 - **onError()**：事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。<br>
 **注意**：在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。
 
 ![Rxjava观察者模式](image/RxjavaEvent.png)
## 基本实现
### 创建观察者（Observer/Subscriber）
```java
Observer<String> observer = new Observer<String>() {    
    @Override    
    public void onNext(String s) {
                Log.d(tag, "Item: " + s);    
                }    
    @Override    
    public void onCompleted() { 
               Log.d(tag, "Completed!");    
    }    
    @Override    
    public void onError(Throwable e) {       
         Log.d(tag, "Error!");   
         }
    };
```
除了 Observer 接口之外，RxJava 还内置了一个实现了 Observer 的抽象类：Subscriber。 Subscriber 对 Observer 接口进行了一些扩展，但他们的基本使用方式是完全一样的：
```java
Subscriber<String> subscriber = new Subscriber<String>()  {
    @Override    
    public void onNext(String s) {        
        Log.d(tag, "Item: " + s);    
        }    
    @Override    
    public void onCompleted() {        
        Log.d(tag, "Completed!");    
        }    
    @Override    
    public void onError(Throwable e) {        
        Log.d(tag, "Error!");    
        }
};              
```
Observer 和 Subscriber 主要有以下两个区别：<br>
1、**onStart()**: 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe() 方法，具体可以在后面的文中看到。<br>
2、**unsubscribe()**: 这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 unsubscribe() 这个方法很重要，因为在 subscribe() 之后， Observable 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 onPause() onStop() 等方法中）调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。

### 创建具体目标（Observable）
Observable就是观察者模式中的具体目标，即被观察的对象。它决定什么时候触发事件以及触发怎样的事件。<br>
下面的代码中，Rxjava使用Observable的create()方法来创建一个具体目标，并在其中定义了一系列的触发事件。其中包括三个onNext()和一个onCompleted();
```java
Observable observable = Observable.create(
    new Observable.OnSubscribe<String>() {    
        @Override    
        public void call(Subscriber<? super String> subscriber){
            subscriber.onNext("Hello");        
            subscriber.onNext("Hi");        
            subscriber.onNext("Aloha");        
            subscriber.onCompleted();    
        }
    });
```
OnSubscribe作为参数被传进create()创建方法中，此时observable对象中就存储了这个事件列表，当observable被订阅之后，OnSubscribe的call方法就会被调用，对观察者进行通知，观察者Subscriber的事件列表中的事件会被一次调用。这样就实现了从目标向观察者进行事件传递。这段代码本身没有什么意义，但是能够说明Rxjava的实现方式。<br>
除了create()方法,RxJava 还提供了一些方法用来快捷创建事件队列:<br>
1. **just(T...)**:
   ```java
    Observable observable = Observable.just("Hello", "Hi", "Aloha");
    // 将会依次调用：
    // onNext("Hello");
    // onNext("Hi");
    // onNext("Aloha");
    // onCompleted();
   ```
2. **from(T[])/from(Iterable<? extends T>)** 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来:
   ```java
   String[] words = {"Hello", "Hi", "Aloha"};
   Observable observable = Observable.from(words);
   // 将会依次调用：
   // onNext("Hello");
   // onNext("Hi");
   // onNext("Aloha");
   // onCompleted();
   ```

###  创建订阅（Subscribe）
创建了 **Observable** 和 **Observer** 之后，再用 **subscribe**() 方法来实现观察者对目标的订阅。
```java
//observer订阅了observable
observable.subscribe(observer);
//另一种写法
observable.subscribe(subscriber);
```
`observable.subscribe(observer)`内部主要进行了以下操作：
1. 调用 Subscriber.onStart() ,进行事件调用之前的准备工作，默认是空。
2. 调用 Observable 中的 OnSubscribe.call(Subscriber) 。在这里，事件发送的逻辑开始运行。从这也可以看出，在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。
3. 将传入的 观察者Subscriber 作为 Subscription 返回。这是为了方便进行 unsubscribe()取消订阅。
![suscribe示意图](image/suscribe示意图.png)

除了 subscribe(Observer) 和 subscribe(Subscriber) ，subscribe() 还支持不完整定义的回调，RxJava 会自动根据定义创建出 Subscriber 。形式如下：
```java
Action1<String> onNextAction = new Action1<String>() {    
    // onNext()    
        @Override    
        public void call(String s) {      
            Log.d(tag, s);   
            }
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {    
    // onError()    
    @Override    public void call(Throwable throwable) {        
        // Error handling    
        }
};
Action0 onCompletedAction = new Action0() {    
    // onCompleted()    
    @Override    public void call() {        
        Log.d(tag, "completed");    
        }
};
// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
observable.subscribe(onNextAction);
// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
observable.subscribe(onNextAction, onErrorAction);
// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```
这段代码中出现的 Action1 和 Action0都是RxJava的接口。 <br>
- **Action0** 是 RxJava 的一个接口，它只有一个方法 call()，这个方法是无参无返回值的；由于 onCompleted() 方法也是无参无返回值的，因此 Action0 可以被当成一个包装对象，将 onCompleted() 的内容打包起来将自己作为一个参数传入 subscribe() 以实现不完整定义的回调。这样其实也可以看做将 onCompleted() 方法作为参数传进了 subscribe()，相当于其他某些语言中的『闭包』。 <br>
- **Action1** 也是一个接口，它同样只有一个方法 call(T param)，这个方法也无返回值，但有一个参数；与 Action0 同理，由于 onNext(T obj) 和 onError(Throwable error) 也是单参数无返回值的，因此 Action1 可以将 onNext(obj) 和 onError(error) 打包起来传入 subscribe() 以实现不完整定义的回调。事实上，虽然 Action0 和 Action1 在 API 中使用最广泛，但 RxJava 是提供了多个 ActionX 形式的接口 (例如 Action2, Action3) 的，它们可以被用以包装不同的无返回值的方法。

### 简单的同步调用应用举例

1. 打印字符串数组<br>
   将字符串数组 names 中的所有字符串依次打印出来：
   ```java
   String[] names = ...;
   Observable.from(names).subscribe(new Action1<String>() {  
             @Override        
             public void call(String name) {
                        Log.d(tag, name);        
            }    
        });

   ```
2. 由id取得图片并显示<br>
   由指定的一个 drawable 文件 id drawableRes 取得图片，并显示在 ImageView 中，并在出现异常的时候打印 Toast 报错：
   ```java
   int drawableRes = ...;
   ImageView imageView = ...;
   Observable.create(new OnSubscribe<Drawable>() {    
                    @Override    
                    public void call(Subscriber<? super Drawable> subscriber){        
                        Drawable drawable = getTheme().getDrawable(drawableRes)); 
                        subscriber.onNext(drawable);        
                        subscriber.onCompleted();    
                        }})
           .subscribe(new Observer<Drawable>() {
                   @Override    
                   public void onNext(Drawable drawable) {        
                       imageView.setImageDrawable(drawable);    
                       }    
                    @Override    
                    public void onCompleted() {    }    
                    @Override    
                    public void onError(Throwable e) {        
                        Toast.makeText(activity, "Error!", Toast.LENGTH_SHORT).show();    
                        }
                    });

   ```
以上两个例子展示了如何创建出 Observable 和 Subscriber ，再用 subscribe()完成订阅关联。一次Rxjava的基本使用就是这个样子。但是事件的发出（call方法被调用）和消费（onNext等事件列表被运行）在同一个线程中，这是一个同步的观察者模式，并没有展现Rxjava异步调用的精髓。『后台处理，前台回调』的异步机制才是Rxjava中最重要的机制，而要实现异步，则需要用到 RxJava 的另一个概念： **Scheduler** 。

## 异步调用

### 线程控制-Scheduler
在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。<br>
在RxJava 中，**Scheduler** 调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler ，它们已经适合大多数的使用场景：
- **Schedulers.immediate():** 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
- **Schedulers.newThread():** 总是启用新线程，并在新线程执行操作。
- **Schedulers.io():** I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
- **Schedulers.computation():** 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
- **AndroidSchedulers.mainThread():** Android专用的线程，指的是在Android的UI线程（主线程）中运行。

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。
* **subscribeOn()**: 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。
* **observeOn()**: 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。
