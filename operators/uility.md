# 工具类操作符

## do

----
实例方法

在输入流发出值是执行一个有副作用的操作，但输出流的值和输入流是完全相同的。

* <font fact="仿宋">_在输入流有值发出时执行一个函数，只要函数运行时没有发生错误输出的值仍然和输入流上的值一样。_</font>

        ---1--------2---------3----|-->

                do(v => console.log(v));

        ---1--------2---------3----|-->

生成一个输入流的镜像，但是这个镜像在发出值之前会执行一个具有副作用的操作，输入流上的所有通知都能安全的传递到输出流上。这个方法非常适合用来调试代码。

注意，这个操作符和subscribe并不一样，它的返回值也是一个流，所有在没有订阅之前，这些副作用的操作是永远不会执行的，并不像subscribe一样会触发流的执行。

参数

| Name           | Type     | Attribute | Description                |
| -------------- | :------: | :-------: | -------------------------: |
| nextOrObserver | Observer | function  | 可选                       |  |
| error          | function | 可选      | 处理输入流上错误通知的函数 |
| complete       | function | 可选      | 处理输入流上结束通知的函数 |

返回值

Observable 会在发出值之前执行副作用的输入流镜像。

示例

    Rx.Observable.fromEvent(document, 'click')
        .do(v => console.log(v))
        .map(v => v.clientX)
        .subscribe(v => console.log(v));

## delay

----
实例方法

指定一个延迟时间，当输入流有上值到达时，输出流将延迟发射这个值，也可以指定一个日期对象，在到达此日期对象指定的值时发射值。

* <font face="仿宋">_以指定的延迟时间发送值。_</font>

        --------a------------b------------|->

                delay

        --------------a------------b------|>

如果此操作符的参数是一个数字，将被当成一个毫秒数，在输入流上的值到达时，输出流将延迟毫秒数指定的时间发送值，也就是说输出流发射值时最低要保持这个延迟时间。

如果提供的参数是一个日期对象，这个流将会在到达这个日期时才开始发射值，这个日期之前在输入流上产生的值不会被发射。

参数

| Name      | Type           | Attribute | Description                                  |
| --------- | :------------: | :-------: | -------------------------------------------: |
| delay     | number 或 Date |           | 延迟发射值的时间间隔或者开始发射值的日期对象 |
| scheduler | Scheduler      | 可选      | 有来提供时间概念的调度器                     |

返回值

Observable 以指定是延迟时间或日期规则发送值。

示例

    Rx.Observable.fromEvent(document,'click')
        .delay(2000)
        .subscribe(v => console.log(v));
----
    Rx.Observable.fromEvent(document,'click')
        .delay(new Date(2018-05-01))
        .subscribe(v => console.log(v));

## delayWhen

----
实例方法

直到另外一条流发出值后才发出输入流上的值。

* <font face="仿宋">_类似于delay，但它的延迟时间是由另外一条流来决定的。_</font>

        -----a--------b--------d----------|-->

        -------x---|->

        ------------------x-|->

        -------------------------------------x-|->

                delayWhen(durationSelector);

        -------a----------b------------------d-|->

此操作符根据另外一条流来决定输入流上的值何时应该被发射。当输入流上有值到达时，传入的函数会被调用，它可以接收输入流的值作为参数，返回一条流，只有当这条流发出值或完成通知时输入流上的值才会在输出流上发出。

此外它接受第二个可选参数-subscriptionDelay，这个参数是一条流，只有当它发出第一个参数或者完成通知时，输入流才开始被订阅。如果没有提供这个参数，输入流会在输出流被订阅的同时被订阅。

参数

| Name                  | Type                         | Attribute | Description                                                                        |
| --------------------- | :--------------------------: | :-------: | ---------------------------------------------------------------------------------: |
| delayDurationSelector | function(value:T):Observable |           | 接受输入流的值作为参数，返回通知流，只有当通知流发出值时输入流的值才会被输出流发出 |
| subscriptionDelay     | Observable                   |           | 决定输入流何时被订阅的流                                                           |

返回值

Observable 延迟发送值的流。

示例

    Rx.Oservable.fromEvent(document, 'click')
        .delayWhen(event => Rx.Observable.interval(Math.random() * 5000))
        .subscribe(x => console.log(x));

## dematerialize

----
实例方法

将发出Notification对象的流拆解为发出Notification对象所代表的值的流。

* <font face="仿宋">_将Notification对象拆解为它实际所代表的值——next、error或complete通知所代表的值。反向方法是materialize_</font>

        -------------{x}-------------{y}-----------{|}--|-->

                    dematerialize

        --------------x---------------y------------|---->

此操作符的操作对象被假设成一条只发出正常通知而不会发出错误通知的流，输入流的值都是持有实际值的元数据，在输出流上实际就是将这些值拆解到相应通知的流。

返回值

Observable 发出输入流上的通知对象拆解后的值的流。

示例

    var notifA = new Rx.Notification('N', 'A');

    var notifB = new Rx.Notification('N', 'B');

    var notifC = new Rx.Notification('E', void 0, new TypeError('x.toUpperCases is not a function'));

    var materialized = Rx.Observable.of(notifA, notifB, notifC);

    var upperCase = materialized.dematerialize();

    upperCase.subscribe(x => console.log(x), e => console.error(e));

## materialize

----
实例方法

将输入流上发出的值全部包装成Notification对象在输出流上发出。

* <font face="仿宋">_将next，error，complete通知的值都包装成Notification对象，然后在输出流上输出。_</font>

        --------x------------y---------z-------|--->

                    materialize

        --------{x}----------{y}-------{z}-----{|}|->

这个操作符将输入流上的正常值、错误值、完成通知全部包装进Notification对象。当输入流上发出完成通知时，输出流上的next通知中也会发出输入流结束的通知，然后输出流本身也将结束。当输入流发出错误通知时，输出流上的next通知也会发出这个错误，然后输出流结束。

这个操作符可以用来产生流上的元数据，可以和dematerialize配合使用。

返回值

Observable<Notification<T>> 发出通知对象的流，这些通知对象包含了从输入流上获取到的元数据。

示例

    var letters = Rx.Observable.of('a', 'b', 13, 'c');

    var upperCase = letters.map(x => toUpperCase());

    var materialized = upperCase.materialize();

    materialized.subscribe(v => console.log(v));

## observeOn

----
实例方法

使用指定的调度器重新发射输入流上的各个通知。

* <font face="仿宋">_在流的外部保证一个调度器被使用。_</font>

此操作符接受一个调度器作为第一个参数，调度器用来调节输入流上值的发射。如果你不能控制给定的输入流的内部调度器，但又想控制它如何发射值，此时可以使用这个操作符。

输出流上发出值和输入流是相同的，只不过使用指定调度器调节了值的发射过程。需要注意的时这并不是说输入流内部的调度器被替换掉了，原来的调度器仍然会被使用，只是当发出值时，它又被传入此操作符的调度器重新又调节了一次。在同步发出大量值的输入流上使用此操作符时会把这条流分解成异步的块。通常为了实现这效果，需要把调度器直接传递给输入流。observeOn只是简单的将通知延迟一些，以确保通知在预期的时间点发出。

此操作符还可以接收第二个参数，它以毫秒为单位指定延迟通知的发送时间。observeOn 与 delay 操作符最主要的区别是它会延迟所有通知，包括错误通知，而 delay 会当源 Observable 发出错误时立即通过错误。 通常来说，对于想延迟流中的任何值，强烈推荐使用 delay 操作符，而使用 observeOn 时，用来指定应该使用哪个调度器来进行通知发送。

参数

| Name      | Type       | Attribute | Description          |
| --------- | :--------: | :-------: | -------------------: |
| scheduler | IScheduler |           | 调度器               |
| delay     | number     | 可选      | 延迟发射值的时间间隔 |

返回值
Observable<T> 对输入流应用了指定的调度器后的流。

示例

    Rx.Observable.interval(1000)
        .observeOn(Rx.Scheduler.animationFrame)
        .subscribe(val => {
            someDiv.style.height = val + 'px';
        });

## subscribeOn

----
实例方法

使用指定的调度器异步的订阅输入流。

    ----a----------------b-----|-->

                subscribeOn(scheduler);

    ====a================b=====|==>

参数

| Name      | Type      | Attribute | Description |
| --------- | :-------: | :-------: | ----------: |
| scheduler | Scheduler |           | 调度器      |

返回值
Observable<T> 修改过的输入流以便使它的订阅发生在指定的调度器上。

示例

## timeInterval

----
实例方法

参数

| Name      | Type      | Attribute | Description |
| --------- | :-------: | :-------: | ----------: |
| scheduler | Scheduler |           | 调度器      |

## timestamp

----
实例方法

参数

| Name      | Type      | Attribute | Description |
| --------- | :-------: | :-------: | ----------: |
| scheduler | Scheduler |           | 调度器      |

## timeout

----
实例方法

参数

| Name      | Type      | Attribute | Description |
| --------- | :-------: | :-------: | ----------: |
| due       | number    |           |             |
| scheduler | Scheduler |           | 调度器      |

## timeoutWith

----
实例方法

参数

| Name           | Type      | Attribute | Description |
| -------------- | :-------: | :-------: | ----------: |
| due            | any       |           |             |
| withObservable | any       |           |             |
| scheduler      | Scheduler |           | 调度器      |

## toArray

返回值

Observable<any[]> | WebSocketSubject<T> | Observable<T>

## toPromise

将输入流转化成符合ES6标准的promise

参数

| Name        | Type               | Attribute | Description                                                                                                    |
| ----------- | :----------------: | :-------: | -------------------------------------------------------------------------------------------------------------: |
| PromiseCtor | PromiseConstructor | 可选的    | Promise的构造函数。如果没有提供的话，它会在 Rx.config.Promise中寻找构造函数，然后回退成原生的Promise构造函数。 |

返回值

Promise<T> 符合ES6的Promise，使用输入流的最后一个值。

示例

    const source = Rx.Observable.of(42)
        .toPromise();

    source.then(v => console.log(v));
