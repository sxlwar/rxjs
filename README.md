# Rxjs概述

## 简介

----

rxjs是一个使用观察者模式来整合异步操作和事件系统的js库，通过一系列可观测的流（observable）将它们串联起来。Observable是这个库的核心类型，此外还包括诸如Observer，Schedulers，Subjects等类型。还包括一些和数组方法类似或经过演化的操作符，用来协助处理数据。

* <font face="仿宋">_可以把 rxjs 想像成一个可以发射事件的 lodash 库。_</font>

响应式设计结合了观察者模式，迭代模式和基于集合的函数式编程风格，从而提供了一种处理事件序列的理想方式。

在rxjs中用来处理异步事件的核心概念包括：

* observable: 代表了未来可能会产生的一系列的值或事件的集合；
* observer: 回调函数的集合，它知道如何去处理observable上产生的值或者事件，当然也包括异常。
* subscription: 代表当前正在订阅的observable，主要作用是用来取消订阅行为。
* operators: 纯函数，以函数式的风格通过各种各样的操作符相互配合对observable上产生的数据集合进处理。
* subject: 相当于一个事件发射器，允许将相同的值传递给多个订阅者。
* schedulers: 协调observable上数据的发射方式。

### 示例

----

在javascript中通常使用以下方式来注册事件：

    var button = document.querySelector('button');

    button.addEventListener('click', () => console.log('Clicked'));
使用rxjs的方式实现如下：

    var button = document.querySelector('button');

    Rx.Observable.fromEvent(button, 'click')
        .subscribe(() => console.log('Clicked'));

#### 纯度

----

使用不纯的函数时，函数可能会改变外部数据的状态，比如：

    var count = 0;

    var button = document.querySelector('button');

    button.addEventListener('click', () => console.log(`Clicked ${++count} times`));

当使用rxjs时，必须把这些状态隔离出去，比如：

    var button = document.querySelect('button');

    Rx.Observable.fromEvent(button, 'click')
        .scan(count => count + 1, 0)
        .subscribe(count => console.log(`Clicked ${count} times`));

这里的scan操作符的行为和数组的reduce方法的行为非常类似，它提供一个初始值给迭代函数，迭代函数运行后的值将作为下一次迭代的初始值。

#### 流

----

rxjs提供了一套非常丰富的操作符来帮助用户控制数据或事件如何在observable中进行流动。假如我们需要控制用户在一个按钮上每秒最多只能点击一次

使用普通的javascript代码:

    var count = 0;

    var rate = 1000;

    var lastClick = Date.now() - rate;

    var button = document.querySelector('button');

    button.addEventListener('click', () => {
        if(Date.now() - lastClick >= rate) {
            console.log(`Clicked ${++count} timers`);
            lastClick = Date.now();
        }
    });

使用rxjs：

    var button = button.querySelector('button');

    Rx.Observable.fromEvent(button, 'click')
        .throttleTime(1000)
        .scan(count => count + 1, 0)
        .subscribe(count => console.log(`Click ${count} times`));

诸如此类对流进行控制的操作符还有：filter, delay, debounceTime, take, takeUntil, distinct, distinctUntilChanged等等。

#### 值

----

你可以对流中的数据进行转换，当我们需要得到每一次鼠标点击时的x轴坐标时，

使用普通的javascript代码:

    var count = 0;

    var rate = 1000;

    var lastClick = Date.now() - rate;

    var button = document.querySelector('button');

    button.addEventListener('click', (event) => {
        if(Date.now() - lastClick >= rate) {
            count += event.clientX;
            console.log(count);
            lastClick = Date.now();
        }
    });

使用rxjs：

    var button = button.querySelector('button');

    Rx.Observable.fromEvent(button, 'click')
        .throttleTime(1000)
        .map(event => event.ClientX)
        .scan((count,clientX) => count + clientX, 0)
        .subscribe(count => console.log(count));

诸如此类可以产生新值的操作符还有：pluck，pairwise，sample等等。

## 可观测序列-Observable

----

可观测序列可以推送多个值，并且它的推送方式是‘懒’的，它满足下面表格中列出的条件

|          | single        | multiple     |
| -------- |:-------------:| ------------:|
| pull     | function      | iterator     |
| push     | promise       | observable   |

下面这个例子中的observable在被订阅时可以立即推送出1，2，3这个三个值，1秒以后推送第四的值4，然后立即结束。

    var observable = Rx.Observable.create(function(observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        setTimeout(() => {
            observer.next(4);
            observer.complete();
        },1000);
    });

为了获取到这个observable上的值， 我们需要对它进行订阅：

    var observable = Rx.Observable.create(function(observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        setTimeout(() => {
            observer.next(4);
            observer.complete();
        },1000);
    });

    console.log('just before subscribe');

    observable.subscribe({
        next: x => console.log('got value ' + x),
        error: err => console.error('something wrong occurred: ' + err),
        complete: () => console.log('done'),
    });

    console.log('just after subscribe);

这段代码执行后将会得到如下结果：

    just before subscribe
    got value 1
    got value 2
    got value 3
    just after subscribe
    got value 4
    done

### pull vs push

----

这里使用 pull 和 push 来描述值的生产者和消费者之间是如何发生联系的，它们是两种完全不同的协议。

在pull的系统中，值的消费决定什么时间从生产者上获取数据，生产者本身并不关心数据什么时间分发给消费者。

每一个javascript函数都可以看作一个 pull 类型的系统。函数可以产生值，但这是通过调用者主动调用函数，函数运行产生出值返回给调用者的方式进行的，所以可以理解为调用者主动去函数上拉取了一值。

ES2015中介绍另外两种 pull 类型的系统，generator函数 和 iterator。对于它们来讲，遍历器对象的 next 方法可以视作值的消费者，通过iterator.next()可以获取到多个值。

|          | Producer               | Consumer           |
|----------|:----------------------:| ------------------:|
|pull      |被动：当被调用时产生值      | 主动：决定何时发起调用 |
|push      |主动：按自身的设置产生值    | 被动：响应接收到的值   |

在push的系统中，生产者决定什么时候发送值给消费者，消费者并知道什么时候可以接收到值。

ES6中的 promise 就是一个非常典型的 push 系统，promise 将 resolve 的结果传递给注册在它内部的回调函数，这与普通的函数有很大的不同，回调函数何时可以接收到数据完全取决于 promise 什么时间向它传递数据。

rxjs的 Observable 也是一种 push 类型的系统，一个 Observable 可以产生多个值，然后推送给它的订阅者。

* Function 是一个 ‘懒’ 的求值过程，只有在被调用时它才会同步的返回一个值给调用者。
* generator 也是一个 ’懒‘ 的求值过种，在遍历的过程中同步的返回0个或多个值给调用者。
* Promise 经过运算后可能产生一个值，当然也可能产生一个错误。
* Observable 也是一个‘懒’的求值过程，当它被订阅后可以同步或者异步的产生出0个或者无限多个值给调用者，这个过程将一直持续到订阅被取消或者流结束。

### Observable 概述

----

很多人认为Observable就是一个事件发射器或可以产生多个值的promise，实际上这种观点是不正确的。在某些情况下Observable的行为可能与事件发射器的行为类似，例如使用Subject来发射值时，但通常情况下它们的行为与事件发射器有很大的不同。

* <font face="仿宋">_可以把 Observable 想像成一个不接受参数的函数，但是它却可以产生出很多的值。_</font>

请思考以下代码：

    function foo() {
        console.log('hello');
        return 42;
    }

    var x = foo.call();

    console.log(x);

    var y = foo.call();

    console.log(y);

运行后的输出：

    "hello"
    42
    "hello"
    42

使用rxjs的方式：

    var foo = Rx.Observable.create(function(observer) {
        console.log('hello');
        observable.next(42);
    });

    foo.subscribe(function (x) {
        console.log(x);
    });

    foo.subscribe(function (y) {
        console.log(y);
    });

运行后的输出依然是：

    "hello"
    42
    "hello"
    42

首先，函数和Observable的求值过程都是‘懒’的，如果你不调用foo函数，console.log('hello')将不会执行，这对于Observable也是一样的，只要不订阅，console.log('hello')同样也不会执行。另外，每一次的订阅（subscribe）和调用（call）都是互不干扰的，两次函数调用会触发两次执行过程，产生两次副作用，同样，两次订阅也会触发两次订阅过程。对于事件发射器来说却不同，它产生的副作用会传递给每一个事件接收者，它并不理会有没有接收者接收事件，都会把事件发射出去，这与Observable有很大的不同。

* <font face="仿宋">_订阅一个Observable与调用一个函数的过程类似。_</font>

你可能会认为Observable都是异步的，事实上并不是这样，我们看一下函数的调用过程：

    console.log('before');
    console.log('foo.call()');
    console.log('after');

执行后的输出：

    "before"
    "hello"
    42
    "after"

然后以同样的顺序，但是使用Observable：

    console.log('before');
    foo.subscribe(function(x) {
        console.log(x);
    });
    console.log('after');

你会发现结果是相同的：

    "before"
    "hello"
    42
    "after"

这个结果证明了foo这个Observable完全是同步执行的，和调用一个函数没有什么两样。

* <font face="仿宋">_Observable传递值的方式可以是同步也可以是异步的。_</font>

那么Observable与函数之间有什么样的区别？**Observable被订阅后可以返回多个值**，这是普通的函数所无法做到的。例如，你无法像下面这样去实现一个函数：

    function foo() {
        console.log('hello');
        return 42;
        return 100; // 这句永远不会执行
    }

因为函数只能返回一个值，return 100; 会被解释器忽略，如果是在typescript中，编译器会告诉你这里有一个错误。然而这对于Observable来说却不是问题：

    var foo = Rx.Observable.create(function (observer) {
        console.log('hello');
        observer.next(42);
        observer.next(100); // '返回‘另外一个值
        observer.next(200); // 同上
    });

    console.log('before');
    foo.subscribe(function(x) {
        console.log(x);
    });
    console.log('after');

上面的代码执行后会以同步的方式输出：

    "before"
    "Hello"
    100
    42
    200
    "after"

当然， 你也可以让它以异步的方式来返回值：

    var foo = Rx.Observable.create(function (observer) {
        console.log('hello');
        observer.next(42);
        observer.next(100); // '返回‘另外一个值
        observer.next(200); // 同上
        setTimeout(() => {
            observer.next(300);
        },1000);
    });

    console.log('before');
    foo.subscribe(function(x) {
        console.log(x);
    });
    console.log('after');

执行后的输出：

    "before"
    "Hello"
    100
    42
    200
    "after"
    300

总结：

* func.call() 表示_以同步的方式返回一个值。_
* observable.subscribe() 表示_以同步或者异步的方式返回一些值。_

### Observable详解

----

我们可以使用Rx.Observable.create方法或者那些可以**创建** Observable 的操作符来创建 Observable，然后使用一个观察者来**观察** Observable，当 Observable 被**订阅**时它就会通过 next/error/complete/ 方法来通知观察者，当观察者放弃观察行为时如何**处理** Observable 运行产生的数据。这四个动作都被编码在一个可观察的实例中，但其中一些动作与其它动作是相关的，如观察和订阅。

使用Observable时要关注的核心问题：

* 如何创建
* 被谁订阅
* 怎么去执行
* 废弃时如何处理

#### 创建Observable

----

Rx.Observable.create方法实际是 Observable 类的构造函数的别名，它接受一个参数——订阅函数。下面这个示例会创建一个observable，它会每秒发一个 string 类型的值‘hi’ 给它的观察者。

    var observable = Rx.Observable.create(function subscribe(observer) {
        var id = setInterval(() => {
            observer.next('hi');
        },1000);
    });

* <font face="仿宋">_可以通过 create 方法来创建 Observable，但实际情况下我们使用更多的是那些可以创建出 Observable 的操作符，例如：from, interval 等。_</font>

上面代码中的 subscribe 函数对于解释 Observable 来说是非常重要的一部分，接下来解释如何去理解它。

#### 订阅Observable

----

上面例子中的observable可以这样被订阅：

    observable.subscribe(x => console.log(x));

observable变量上有 subscribe 方法，同样 Observable.create(function subscribe(observer) { ... }) 也有subscribe 方法，这并不是一个巧合。在库的实现中，这两个subscribe是不一样的，但在实际使用中你可以认为它们在概念上是相等的。

这个示例很好的展示了为什么订阅行为在多个订阅者之间是不会共享的。当我们使用一个观察者来订阅 observable 时，传入 create 方法的 subscribe 函数就会为这个观察者运行一次。每次调用 observable.subscribe 都会为给定的观察者触发它自己的独立设置。

* <font face="仿宋">_订阅Observable和执行一个函数很类似，都可以提供一个回调函数来接收将要产生的值。_</font>

这个过程与诸如 addEventListener/removeEventListener 等事件处理API有很大的区别。在 observable.subscribe 的过程中，给定的观察者并没有注册成为 Observable 上的监听者。Observable 甚至不需要维护一个观察者名单。

通过 subscribe 这种简单的方式就可以使一个 Observable 开始执行，并且将产生的数据或事件传递给当前 Observable 环境上的观察者。

#### 执行Observable

----

Observable.create(function subscribe(observer) { ... })内部的代码代表了 Observable 的执行，一个‘懒’执行过程——仅在有观察者订阅它的时候执行，当它被执行后就可以以同步或异步的方式不断的生产值。

Observable 执行后可以产生三种类型的值：

* ’Next‘：发送一个**正常**的值，比如String， Number 或者 Object 等。
* ‘Error’：发送一个javascript的错误或者抛出异常。
* ’Complete‘：停止发送值。

Next类型的通知是最重要也是最常用的一种，它代表了将要传递给观察者的数据。Error  和 Complete 类型的通知在整个Observable的执行过程中只可以发生一次，并且它们是互斥的，只有一个可以发生。

以正则表达式来表述三者之间的关系的话，应该是：

    next*(error|complete)?

* <font face="仿宋">_在Observable执行的过程中，可以发生0个或无限多个Next类型的通知，但是当 Error 或 Complete 类型的通知中有一个发生后，Observable将不会再传递任何数据。_</font>

下面的代码展示了一个可以产生三个 Next 类型通知的 Observable，最后发送一个 Complete 通知：

    var observable = Rx.Observable.create(function subscribe(observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.complete();
    });

Observable 将严格遵守上述规则，所以试图在Observable完成后继续传递值的行为是无效的。

    var observable = Rx.Observable.create(function subscribe(observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.complete();
        observer.next(4); // 不会传递成功
    });

如果Observable在执行过程中发生异常，我们可以在subscribe函数内部使用 try/catch 块来捕获它：

    var observable = Rx.Observable.create(function subscribe(observer) {
        try {
            observer.next(1);
            observer.next(2);
            observer.next(3);
            observer.complete();
        } catch(error) {
            observer.error(error); // 传递错误
        }
    });

#### 处理Observable

----

由于 Observable 可能产生出无限多个值，但观察者可能在某时间点想停止观察行为，这就需要有一种方法来取消 Observable 的执行。由于每个执行只对应一个观察者，一旦观察者完成接收值，它就需要知道如何停止执行，以避免浪费计或内存资源。

当我们调用 observable.subscribe 时，观察者会被附加在新创建的可观察上下文中，同时这次调用会返回一个对象，它就是 Subscription 。

    var subscription = observable.subscribe(x => console.log(x));

Subscription 代表了正在执行的上下文，它拥有一个可以取消 Observable 执行的方法——unsubscribe。调用这个方法 subscription.unsubscribe() 就可以取消执行。

    var observable = Rx.Observable.from([10,20,30]);
    var subscription = observable.subscribe(x => console.log(x));
    // 一段时间以后
    subscription.unsubscribe();

* <font face="仿宋">_当你对一个 Observable 进行订阅时，就会获得一个代表当前执行的 Subscription ，只需要调用它的 unsubscribe 方法就可以取消订阅。_</font>

当我们使用 Observable.create 方法来创建 Observable 时，必须定义当取消订阅时如何去释放执行时的资源，此时可以返回一个自定义的 unsubscribe 函数。

我们可以给之前的例子添加上它的 unsubscribe 方法：

    var observable  Rx.Observable.create(function subscribe(observer) {
        var intervalID = setInterval(() => {
            observer.next('hi');
        },1000);

        return function unsubscribe() {
            clearInterval(intervalID);
        }
    });

与 observable.unsubscribe 一样，我们可通过调用 unsubscribe 方法对这个流取消订阅，这两个 unsubscribe 在概念上是完全相同的。

事实上在这个例子中，假如我们把响应式的外壳剥离的话，它完全就是一个普普通通的javascript函数：

    function subscribe(observer) {
        var intervalID = setInterval(() => {
            observer.next('hi');
        },1000);

        return function unsubscribe() {
            clearInterval(intervalID);
        }
    }

    var unsubscribe = subscribe({next: (x) => console.log(x)});

尽管如此，我们依然有理由去使用Rx这种响应式的编程，通过 Observable，Observer，Subscription，配合各种操作符来实现更高效安全的数据处理。

## 观察者——Observer

----

观察者实际就是可观察序列——Observable的消费者。Observer 会设置一些回调函数来对应 Observable 的各个通知类型（next/error/complete)，从这些通知中接收值，然后对它们进行处理。

这是一个很典型的 Observer：

    var observer = {
        next: x => console.log('Observer got a next value: ' + x),
        error: err => console.error('Observer got an error: ' + err),
        complete: () => console.log('Observer got a complete notification')
    };

我们可以使用它来观察一个 Observable：

    observable.subscribe(observer);

* <font face="仿宋">_观察者就是一个简单对象，拥有三个方法，每一个方法对应处理可观察序列上的相应类型的通知。_</font>

Rxjs中的观察者允许只实现这三个方法中的某几个，这对 Observable 的执行过程不会产生影响，仅仅是被忽略方法的对应通知无法得到处理，因为观察者想要处理相应通知上的数据时必须实现对应的方法。

下面这个示例的 Observer 就没有实现处理 complete 方法：

    var observer = {
        next: x => console.log('Observer got a next value: ' + x),
        error: err => console.error('Observer got an error: ' + err),
    };

当需要订阅一个 Observable 时，可以只传入一个回调函数作为参数，而不必传入观察者对象：

    observable.subscribe(x => console.log('Observer got a next value: ' + x));

实际上在库的内部实现中会根据传入的参数为我们创建出一个 Observer 对象，此时，传入的第一个函数作为next方法的回调，第二个是error回调，第三个是complete回调。也就是说我们可以这样去订阅一个 Observable ：

    observable.subscribe(
        x => console.log('Observer got a next value: ' + x),
        err => console.error('Observer got an error: ' + err),
        () => console.log('Observer got a complete notification')
    );

## Subscription

----

Subscription是一个代表着当前正在执行的 Observable 的对象。它有一个非常重要的方法 unsubscribe ，这个方法不接受任何参数，仅仅是用来释放 Observable 执行时的资源。在旧版本的Rxjs中，它也叫做 Disposable 。

    var observable = Rx.Observable.interval(1000);
    var subscription = observable.subscribe(x => console.log(x));
    // 执行一段时间后

    // 下面的操作将会取消上面已经订阅的 Observable 的执行。
    subscription.unsubscribe();

* <font face="仿宋">_Subscription本质上可以只有一个 unsubscribe 方法，用来取消已经订阅的 Observable 的执行。_</font>

Subscription 可以被组合在一起，这样只要调用一次 unsubscribe 方法就可以将多个 Observable 的执行取消掉。我们可以通过 subscription 的 add 方法将它们添加在一起。

    var observable1 = Rx.Observable.interval(400);
    var observable2 = Rx.Observable.interval(300);

    var subscription = observable1.subscribe(x => console.log('first: ' + x));

    var childSubscription = observable2.subscribe(x => console.log('second: ' + x));

    subscription.add(childSubscription);

    setTimeout(() => {
        // 取消上面的两次订阅
        subscription.unsubscribe();
    },1000);

执行后将输出：

    second:0
    first:0
    second:1
    first:1
    second:2

当然，可以添加，就可以移除，Subscription同样拥有一个remove方法，用来从一组subscription 中移除指定的子 subscription。

## Subject

----

Subject 是 Observable 的一种特殊类型，它允许把值同时传播给多个 Observer，也就是说它是多播的。相比之下，普通的Observable 是单播的，每一个 Observer 在订阅 Observable 时都有其独立的执行上下文。

* <font face="仿宋">_Subject 类似于 Observable ，但它可以把值同时广播给多个观察者。它也类似于一个事件发射器，因此它会维护一个属于自己的监听者列表。_</font>

**每一个 Subject 都是一个 Observable**。你可以提供一个订阅者来订阅 Subject，从而在它上面取值。从观察者的角度来看，并不能区分出数据是从 Subject 上获取的还是从 Observable 上获取的。

在 Subject 的内部实现中，并不会产生新的执行上下文来传递数据，它仅仅是简单的将 Observer 注册在自己的监听者列表中，这各其它的库或语言中添加事件的机制是类似的。

**每一个 Subject 都是一个 Observer**。每一个 Subject 上都有自己的 next，error，complete方法。当需要给 Subject 上添加一个值时，只要调用它的next方法，接下来它就会把这个值广播给注册在监听者列表中的多个监听者。

下面的例子中，我们给 Subject 添加了两个观察者，然后给它添加一些值：

    var subject = new Rx.Subject();

    subject.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    subject.subscribe({
        next: v => console.log('observerB: ' + v)
    });

    subject.next(1);
    subject.next(2);

执行后的输出：

    observerA: 1
    observerB: 1
    observerA: 2
    observerB: 2

由于 Subject 也是一个 Observer ，所以你可以把他传递给一个 Observable 的subscribe方法：

    var subject = new Rx.Subject();

    subject.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    subject.subscribe({
        next: v => console.log('observerB: ' + v)
    });

    var observable = Rx.Observable.from([1,2,3]);

    observable.subscribe(subject); // 通过 Subject 来订阅这条 Observable

执行后输出：

    observerA: 1
    observerB: 1
    observerA: 2
    observerB: 2
    observerA: 3
    observerB: 3

通过上面的方法，我们基本上就借助 Subject 可以把一个单播的 Observable 转换成多了多播的。这个示例仅仅是演示了一种将一个执行上下文分享给多个观察者的方法。

在 Subject 类型下，还有一些比较特殊的 Subject 类型：BehaviorSubject，ReplaySubject，AsyncSubject。

### 多播的Observable

----

一个‘多播’的 Observable 可以借助于 Subject 实现将数据通知给多个观察者，然而单播的 Observable 仅仅只能把通知发送给一个观察者。

* <font face="仿宋">_多播的 Observable 会使用 Subject 作为钩子来实现将同一个 Observable 的执行传递给多个观察者。_</font>

这就是那些多播的操作符实现时的真实情况：观察者订阅底层的一个 Subject，使用这个 Subject 来订阅产生源数据的 Observable。

下面这个例子和之前那个使用 subject 来订阅 Observable 的例子非常类似：

    var source = Rx.Observable.from([1,2,3]);
    var subject = new Rx.Subject();
    var multicasted = source.multicast(subject);

   // 使用 subject.subscribe({...}) 实现
   multicasted.subscribe({
       next: v => console.log('observerA: ' + v)
   });
   multicasted.subscribe({
       next: v => console.log('observerB: ' + v)
   });

    //使用 source.subscribe(subject) 实现
   multicasted.connect();

上面代码中的multicast方法返回的 Observable 看起来像一个普通的 Observable，但是当它被订阅时，执行的方式却和 Subject 一样是多播的，同时这个 Observable 还有一个connect方法，可以将多个流串联在一起。

何时调用这个connect方法非常重要，因为它决定了这条可以共享的流什么时间开始执行。由于connect方法在内部执行了 source.subscribe(subject) ，因此它会返回一个 Subscription ，可以通过它来取消这个共享的 Observable 的执行。

#### 引用计数

----

通过手动调用connect方法来得到 Subscription 多少显得有笨重，通常情况下，我们希望当第一个观察者抵达时可以自动的调用connect方法，当最后一个观察者取消订阅时自动的取消这个共享 Observable 的执行。

请考虑实现如以下列表所概述的订阅：

1. 第一个观察者订阅多播流。
2. 多播流被connect。
3. 0 被传递给第一个订阅者。
4. 第二个观察者订阅这条多播流。
5. 1 被传递给第一个订阅者。
6. 1 被传递给第二个订阅者。
7. 第一个订阅者取消订阅多播流。
8. 2 被 传递给第二个订阅者。
9. 第二个订阅者取消订阅多播流。
10. 取消这条多播流的订阅。

为了达到这个效果，我们需要显式的调用connect方法：

    var source = Rx.Observable.interval(500);
    var subject = new Rx.Subject();
    var multicasted = source.multicast(subject);
    var subscription1, subscription2, subscriptionConnect;

    subscription1 = multicasted.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    // 在这里我们需要显示的调用connect方法以使第一个订阅者可以开始接收值
    subscriptionConnect = multicasted.connect();

    setTimeout(() => {
        subscription2 = multicasted.subscribe({
            next: v => console.log('observerB: ' + v)
        });
    }, 600);

    setTimeout(() => {
        subscription1.unsubscribe();
    },1200);

    // 在这里我们需要把多播流的订阅取消掉，因为从此以后再也没有订阅者订阅它了。
    setTimeout(() => {
        subscription2.unsubscribe();
        subscriptionConnect.unsubscribe();
    },2000);

我们可以通过使用 ConnectableObservable 的refCount方法来避免显式的调用connect方法，这个方法会返回一个 Observable 用来追踪当前有多少订阅者正在订阅多播流。当订阅者的数量从0增加到1时，它帮助我们调用connect方法以开始订阅，当订阅者的数量从1变为0时，它帮助我们取消订阅以停止多播流的继续执行。

* <font face="仿宋">_refCount使得多播流在第一个订阅者到达时开始执行，在最后一个订阅者离开后停止执行。_</font>

请看下面的例子：

    var source = Rx.Observable.interval(500);
    var subject = new Rx.Subject();
    var refCount = source.multicast(subject).refCount();
    var subscription1, subscription2, subscriptionConnect;

    // 这次调用会执行connect方法
    console.log('observerA subscribed');
    subscription1 = refCounted.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    setTimeout(() => {
        console.log('observerB subscribed');
        subscription2 = refCounted.subscribe({
            next: v => console.log('observerB: ' + v)
        })
    },600);

    setTimeout(() => {
        console.log('observerA unsubscribed');
        subscription1.unsubscribe();
    }, 1200);

    // 这里会调用多播流的unsubscribe方法
    setTimeout(() => {
        console.log('observerB unsubscribed');
        subscription2.unsubscribe();
    },2000);

执行后的输出：

    observerA subscribed
    observerA: 0
    observerB subscribed
    observerA: 1
    observerB: 1
    observerA unsubscribed
    observerB: 2
    observerB unsubscribed

refCount方法仅存在于 ConnectableObservable 上，它返回的是一个新的 Observable 而不是另一个 ConnectableObservable 。

### BehaviorSubject

----

BehaviorSubject 是 Subject 的一个变种上，关于它的一个重要概念是’当前值‘。它会把最后一次发送给订阅者的值保存下来，当另外一个订阅者开始订阅时，它会把这个值立即发送给新的订阅者。

* <font face="仿宋">_BehaviorSubject 对于表示随时间变化的值是非常有用的。例如，表示生日的事件流可以用 Subject，但是一个人的年龄的事件流可以用 BehaviorSubject。_</font>

下面的例子中，BehaviorSubject初始化了一个值0，当第一个订阅者开始订阅时，这个值被发送出去。接下来当第二个订阅者开始订阅时，将会接收到值2，即使这个值已经被发送过了。

    var subject = new Rx.BehaviorSubject(0);

    subject.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    subject.next(1);
    subject.next(2);

    subject.subscribe({
        next: v => console.log('observerB: ' + v)
    });

    subject.next(3);

执行后的输出：

    observerA: 0
    observerA: 1
    observerA: 2
    observerB: 2
    observerA: 3
    observerB: 3

### ReplaySubject

----

ReplaySubject 和 BehaviorSubject 很类似，它们都可以把过去的值发送给新的订阅者，不同的是它可以把 Observable 执行时产生的一部分值记录下来。

* <font face="仿宋">_ReplaySubject 记录 Observable 执行时的多个值，并将它们传递给新加入的订阅者。_</font>

在创建 ReplaySubject 时，我们可以指定需要重复发送的值的数量：

    var subject = new Rx.ReplaySubject(3); // 传递3个值给新加入的订阅者

    subject.subscribe({
        next: (v) => console.log('observerA: ' + v)
    });

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
        next: (v) => console.log('observerB: ' + v)
    });

    subject.next(5);

执行后的输出：

    observerA: 1
    observerA: 2
    observerA: 3
    observerA: 4
    observerB: 2
    observerB: 3
    observerB: 4
    observerA: 5
    observerB: 5

还可以指定一个以毫秒为单位的时间，配合记录的数量共同决定究竟有个多少个值需要被重复发送。

下面的这个例子，使用了一个比较大的数量，但是时间上只限制了只发送500毫秒内的值：

    var subject = new ReplaySubject(100, 500/* windowTime */);

    subject.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    var i = 1;
    setInterval(() => subject.next(i++), 200);

    setTimeout(() => {
        subject.subscribe({
            next: v => console.log('observerB: ' + v)
        });
    },1000);

第二个订阅者只能接收到最近的500毫秒内发出的值，下面是执行后的输出，

    observerA: 1
    observerA: 2
    observerA: 3
    observerA: 4
    observerA: 5
    observerB: 3
    observerB: 4
    observerB: 5
    observerA: 6
    observerB: 6
    ...

### AsyncSubject

----

它也是 Subject 的一个变种，AsyncSubject仅在流执行结后把最后一个值发送给它的订阅者。

    var subject = new Rx.AsyncSubject();

    subject.subscribe({
        next: v => console.log('observerA: ' + v)
    });

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
        next: v => console.log('observerB: ' + v);
    });

    subject.next(5);
    subject.complete();

执行后的输出：

    observerA: 5
    observerB: 5

AsyncSubject 的行为与last操作符的行为非常相似，都是在等待complete执行再发送一个值。

## 操作符

----

虽然 Observable 是构建rxjs的基石，但是真正精彩的部分应该是操作符。操作符的组合使用使得以声明式的方式解决复杂的异步场景变得非常简单，它构成rxjs必不可少的一部分。

### 什么是操作符

----

操作符是 Observable 类上的方法，例如： .map(...)，.filter(...)，.merge(...)等。当我们调用它们时，并不会去改变已有 Observable 实例，取而代之的是创建一个全新的 Observable ，它是订阅逻辑是基于第一个 Observable 的。

* <font face="仿宋">_操作符是一个纯函数，它不会改变已有的 Observable 实例，而是基于当前的Observable 创建一个新的 Observable。_</font>

理解操作符时非常重要的一点是时刻谨记它是一个纯函数，它接受一个 Observable 作为输入然后生成一个新的 Observable 作为输出。当订阅输出 Observable 的同时也就订阅了输入 Observable 。

下面的例子中，我们自定义了一个操作函数，它将输入的每一个值都乘以10：

    function multiplyByTen(input) {
        var output = Rx.Observable.create(function subscribe(observer) {
            input.subscribe({
                next: v => observer.next(10 * v),
                error: err => observer.error(error),
                complete: () => observer.complete()
            });
        });

        return output;
    }

    var input = Rx.Observable.from([1,2,3,4]);
    var output = multiplyByTen(input);
    output.subscribe(x => console.log(x));

执行后的输出：

    10
    20
    30
    40

注意：我们只是订阅了输出-——output，但这同时导致input被订阅，这称之为’操作符的链式订阅‘。

### 实例的操作符 VS 静态操作符

----

通常在提到操作符时，我们指的是 Observable 实例上的操作符。假如上面例子中我们定义的操作函数 multiplyByTen 是一个操作符的话，它看起来应该是这样：

    Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
        var input = this;
        return Rx.Observable.create(function subscribe(observer) {
            input.subscribe({
                next: v => observer.next(10 * v),
                error: err => observer.error(err),
                complete: () => observer.complete()
            });
        });
    }

* <font face="仿宋">_实例操作符是使用this关键字来推断输入 Observable 的函数。_</font>

在这里输入的 Observable 不再通过 multiplyByTen 的参数获得，而是通过this关键字来获取，这就允许我们可以像下面这样链式的调用：

    var observable = Rx.Observable.from([1,2,3,4]).multiplyByTen();

    observable.subscribe(x = console.log(x));

除了实例操作符，还有静态操作符，它们是直接定义在 Observable 类上的静态方法。静态操作符不会使用this关键字来推断输入，它的输入完全依赖于输入的参数。

* <font face="仿宋">_静态操作符时附加在 Observable 类上的静态函数，通常用来从头创建 Observable。_</font>

常见的静态操作符都是一些可创建类型的操作符，与通常情况下操作符接收输入流，输出输出流不同，它们大多接收一些非 Observable 类型的参数，例如一个数字，然后创建出一条流。

    var observable = Rx.Observable.interval(1000 /* 毫秒数 */);

另外的例子就是create，我们在之前的例子中已经多次使用过了。

然后，静态操作符并非只是那些简单创建流的操作符，一些组合类的操作符也可以有它的静态类型，例如：merge，combineLatest，concat等。这样做的意义在于我们接收多个流作为输入而不仅仅是一个。

    var observable1 = Rx.Observable.interval(1000);
    var observable2 = Rx.Observable.interval(400);

    var merge = Rx.Observable.merge(observable1, observable2);

#### 弹珠图

----

单纯靠文字可能还不足解释的清楚操作符是如何工作的，许多操作符是与时间相关的，它们可能以不同的方式来影响值的发射，比如：delay，throttle，sample等。对于这些操作符来说图表的形式会更加直观。弹珠图可以模拟出操作符是如何工作的，可以直观的表现出输入流，操作符，参数与输出流之间的联系。

下面来详细解释弹珠图各部分的含义：

    // 这条从左到右的横线代表随时间的推移，输入流的执行过程。
    // 横线上的值代表从流上发射出的值
    // 横线尾部的竖线代表complete通知执行的时间点，表示这条流已经成功的执行完成。
    ----------4------6--------------a-------8-------------|---->

                multipleByTen // 使用的操作符

    // 这条从左到右的横线代表经过操作符转换后的输出流。
    // 横线尾部的X代表在这个时间点上流发生了错误，至此之后不应该再有 Next 通知或 Complete 通知从流上发出。
    ---------40-----60--------------X--------------------------->

在整个文档中，我们都使用这种弹珠图来解释操作符是如何工作的。这种弹珠图在其它场景中也是非常有用的，比如在白板演示或者单元测试中。

#### 如何选择合适的操作符

----

如果你明确自己需要解决的问题，但是不清楚应该使用哪一个操作符，可以在官网的 Manual > Operators > Choose an operator 中通过选择符合问题的描述来找到你所需要的操作符。

#### 操作符类别

----

为了解决不同的问题，rxjs针对性的设计了各种各样的操作符，大体上可以分为几类：创建型、转换型、过滤型、组合型、多播型、错误处理型、工具类等等。

关于操作符，将会在专门的文档中进行解释。

### Scheduler

----

Scheduler决定了subscription什么时候开始以及什么时候开始分发值。它由3个部分组成：

* 一个Scheduler就是一段数据结构：它知道如何去存储数据，以及基于优先级或其它标准来排列任务的顺序。
* 一个Scheduler就是一个执行环境：决定什么时间在什么样的环境下去执行任务， 如何是立即执行还是在回调中执行，亦或是在定时器到达时，还是下一次事件循环开始时执行。
* Scheduler有自己的虚拟时钟：被调度任务的执行时间只由虚拟时钟决定。只会它的虚拟时钟的环境中执行。

### Scheduler类型

----

* null： 按正常的规则发送值。
* queue: 按当前事件队列中的顺序来发送值。
* asap：按更小的单元上的事件队列中的顺序发送值。
* async: 按异步顺序来发送值。

可以传入scheduler的操作符：

* bindCallback
* bindNodeCallback
* combineLatest
* concat
* empty
* from
* fromPromise
* interval
* merge
* of
* range
* throw
* timer
