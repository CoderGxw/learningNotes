# Rxjava
项目地址：https://github.com/ReactiveX/RxAndroid

- [Rxjava](#Rxjava)
  - [定义](#%E5%AE%9A%E4%B9%89)
  - [基本概念](#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
    - [四个基本概念](#%E5%9B%9B%E4%B8%AA%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
    - [事件回调方法](#%E4%BA%8B%E4%BB%B6%E5%9B%9E%E8%B0%83%E6%96%B9%E6%B3%95)
  - [基本实现](#%E5%9F%BA%E6%9C%AC%E5%AE%9E%E7%8E%B0)
    - [创建观察者（Observer/Subscriber）](#%E5%88%9B%E5%BB%BA%E8%A7%82%E5%AF%9F%E8%80%85ObserverSubscriber)
    - [创建具体目标（Observable）](#%E5%88%9B%E5%BB%BA%E5%85%B7%E4%BD%93%E7%9B%AE%E6%A0%87Observable)
    - [创建订阅（Subscribe）](#%E5%88%9B%E5%BB%BA%E8%AE%A2%E9%98%85Subscribe)
    - [简单的应用举例](#%E7%AE%80%E5%8D%95%E7%9A%84%E5%BA%94%E7%94%A8%E4%B8%BE%E4%BE%8B)

## 定义

>Rxjava：一个基于观察者模式，用来实现异步操作的库

## 基本概念 
### 四个基本概念
Rxjava有四个基本概念，分别是Observable (具体目标，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。其中observable和observer通过subscribe()实现订阅关系, Observable 可以在需要的时候发出事件来通知 Observer。<br>

### 事件回调方法
 RxJava 的事件回调方有三个：<br>
 1.**onNext()**:普通事件。相当于Onclick()/onEvent()<br>
 2.**onCompleted()**:事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志。<br> 
 3.**onError()**：事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。<br>
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
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {    
    @Override    
    public void call(Subscriber<? super String> subscriber) {        subscriber.onNext("Hello");        
    subscriber.onNext("Hi");        
    subscriber.onNext("Aloha");        
    subscriber.onCompleted();    }
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
**Action0** 是 RxJava 的一个接口，它只有一个方法 call()，这个方法是无参无返回值的；由于 onCompleted() 方法也是无参无返回值的，因此 Action0 可以被当成一个包装对象，将 onCompleted() 的内容打包起来将自己作为一个参数传入 subscribe() 以实现不完整定义的回调。这样其实也可以看做将 onCompleted() 方法作为参数传进了 subscribe()，相当于其他某些语言中的『闭包』。 <br>
**Action1** 也是一个接口，它同样只有一个方法 call(T param)，这个方法也无返回值，但有一个参数；与 Action0 同理，由于 onNext(T obj) 和 onError(Throwable error) 也是单参数无返回值的，因此 Action1 可以将 onNext(obj) 和 onError(error) 打包起来传入 subscribe() 以实现不完整定义的回调。事实上，虽然 Action0 和 Action1 在 API 中使用最广泛，但 RxJava 是提供了多个 ActionX 形式的接口 (例如 Action2, Action3) 的，它们可以被用以包装不同的无返回值的方法。
### 简单的应用举例
