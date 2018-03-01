# 创建型操作符

## bindCallback

----
静态方法

将回调API转化成一个返回值为Observable的函数。

* <font face="仿宋">_给它一个签名为f(x, callback)的函数f,返回一个函数g, 调用'g(x)'的时候会返回一个 Observable。_</font>

bindCallback不能称之为一个操作符，因为它的输入和输出都不是一个 Observable。它的输入必须是一个函数，这函数的最后一个参数必须是一个回调函数，当输入函数执行完成时调用此回调函数。它的输出也是一个函数，这函数的参数除了最后一个不是回调函数外，其它的和输入函数完成相同。当输出函数被调用时，它会返回一个流，如果输入函数调用回调函数时只传入了一个参数，这个参数将会在流上输出，如果输入函数调用回调函数时传入了多个参数，这些参数会以数组的形式在流上输出。

非常重要的一点是，在输出函数返回的流被订阅之前，输入函数不会执行，也就是说如果输入函数会发出ajax请求，那么这个请求只会在每一次有人订阅输出函数返回的流时才执行，在此之前并不会发请求。

此外，还可以传入一个可选的selector函数作为参数，这个函数的入参和回调函数相同，而返回值将作为最终流上输出的值。在多个参数的情况下，selector函数会直接使用这些参数，而不是以数组的形式使用。也就是说可以把默认的selector函数想像成在多参数情况下以数组形式返回参数，在单参数情况下直接返回参数的函数。

最后一个可选参数是 Scheduler 用来控制当流被订阅时输入函数在什么时间触发，以及流上的值什么时间被发出。默认情况下，订阅流会同步触发输入函数的执行。如果使用 Scheduler.async，那么将会延迟输入函数的触发时机，类似于使用 setTimeout(..., 0)时的情形一样，意味着所有正在执行的函数都将在调用输入函数前就执行完成。

默认情况下传入回调函数的结果在输入函数被调用后会立即被发射，特别是如果回调也是同步调用的话，Observable也会同步的调用next方法。如果希望延迟这些调用可以使用 Scheduler.async，这样可以保证异步的调用输入函数以避免乱象。

需要注意的是，输出函数返回的Observable只能发出一次值，然后完成。即使输入函数多次调用回调函数，第二次以及之后的调用都不会出现在流中。如果需要监听多次的调用，可以使用fromEvent或者 fromEventPattern。

如果输入函数依赖上下文(this)，该上下文将被设置为输出函数在调用时的同一上下文。特别是如果输入函数被当作是某个对象的方法进行调用时，为了保持同样的行为，建议将输出函数的上下文设置为该对象。

如果输入函数以 node 的方式(第一个参数是可选的错误参数用来标示调用是否成功)调用回调函数，bindNodeCallback 提供了方便的错误处理，也许是更好的选择。 bindCallback 不会区别对待这些方法，错误参数(是否传递) 被解释成正常的参数。

| Name      | Type      | Attribute | Description                                                |
| --------- | :-------: | :-------: | ---------------------------------------------------------: |
| func      | function  |           | 使用回调函数作为最后一个参数的函数                         |
| func      | function  | 可选      | 从回调函数上接收值，然后将它们映射成一个新的值给输出流使用 |
| scheduler | Scheduler | 可选      | 使用scheduler控制何时调用回调函数                          |

返回值

function 返回一个 Observable的函数。

示例

    var getJSONAsObservable = Rx.Observable.bindCallback(jQuery.getJSON);

    var result = getJSONAsObservable('/my/url/');

    result.subscribe(v => console.log(x), e => console.error(e));

----

    someFunction((a, b, c) => {
        console.log(a);
        console.log(b);
        console.log(c);
    })

    const boundSomeFunction = Rx.Observable.bindCallback(someFunction);

    boundSomeFunction().subscribe(values => console.log(values));

----

    function add(a, b, c) {
        console.log('input function called');

        return fn => fn(a, b, c);
    }

    const boundSomeFunction = Rx.Observable.bindCallback(add(1,2,3),(a,b,c) => a + b + c);

    boundSomeFunction().subscribe(v => console.log(v));

----

    function iCallMyCallbackSynchronously(cb) {
        cb();
    }

    const boundSyncFn = Rx.Observable.bindCallback(iCallMyCallbackSynchronously);
    const boundAsyncFn = Rx.Observable.bindCallback(iCallMyCallbackSynchronously, null, Rx.Scheduler.async);

    boundSyncFn().subscribe(() => console.log('I was sync!'));
    boundAsyncFn().subscribe(() => console.log('I was async!'));
    console.log('This happened...');

----

    const someObject = {
        methodWithCallback(fn) {
            fn();
        }
    }

    const boundMethod = Rx.Observable.bindCallback(someObject.methodWithCallback);

    boundMethod.call(someObject) // make sure methodWithCallback has access to someObject
        .subscribe(subscriber);

## bindNodeCallback

----
静态方法

将一个node-js风格的API转换成一个发出 Observable 的函数。

* <font face="仿宋">_和bindCallback一样，只不过期望传入一个node-js风格的回调函数。_</font>

bindNodeCallback不能称之为一个操作符，因为它的输入和输出都不是一个 Observable。它的输入必须是一个函数，这函数的最后一个参数必须是一个回调函数，当输入函数执行完成时调用此回调函数。这个回调函数需要符合node-js的风格，也就是说第一个参数必须是一个错误对象，标识执行过程是否发生了错误，如果传入了这个参数，说明调用发生了错误。它的输出也是一个函数，这函数的参数除了最后一个不是回调函数外，其它的和输入函数完成相同。当输出函数被调用时，它会返回一个流。如果输入函数在调用它的回调函数时传递了错误参数，这个错误将会在流上输出。如果没有传递错误参数，流上将会输出第二个参数，如果有更多的参数，流上将会以数组的形式输出这些参数，当然，错误参数除外。

此外，还可以传入一个可选的selector函数作为参数，用来将值转化成希望在流上输出的值，而不是直接输出通常情况下的回调参数。这和bindCallback的selector函数类似，但是错误参数永远不会被传入到这个函数中。

注意在输出产生的流被订阅之前输入函数是不会被调用的。默认情况下输入函数的调用在发生订阅后同步发生，但也可以通过调度器来改变这种行为模式。调度器可以控制来自于回调函数的值何时在流上输出。这和在bindCallback上使用调度器时是相同的。

和bindCallback一样它也可以设置函数运行时的执行环境。

输出函数返回的Observable只能发出一次值，然后完成。即使输入函数多次调用回调函数，第二次以及之后的调用都不会出现在流中。如果需要监听多次的调用，可以使用fromEvent或者 fromEventPattern。

bindNodeCallback也可以在非node环境中使用，因为node-js风格的回调仅仅是一种书写约定，不管是哪种环境下，只要回调的书写风格符合node-js的回调风格，bindNodeCallback就可以安全的处理它。

传递给回调函数的第一个错误对象没必要非得是javaScripte内置的Error对象的实例，事实上，它甚至可以不是一个对象。错误参数这里只是用来标识是否有错误存在，一切可以转化的真值的参数都代表着有错误发生，因此它可以是一个布尔真值，一个非0数字，非空字符串等，所有这些情况都会使用最终的流输出一个错误。这意味着当使用bindNodeCallback 的时候通常形式的回调函数都会触发失败。如果你的 Observable 经常发生你预料之外的错误，请检查下回调函数是否是 node-js 式的回调，如果不是，请使用bindCallback替代。

注意，即使错误参数出现在回调函数中，但是它的值是假值，它仍然不会出现在Observable的发出数组或者选择函数中。

| Name      | Type      | Attribute | Description                                                |
| --------- | :-------: | :-------: | ---------------------------------------------------------: |
| func      | function  |           | 使用回调函数作为最后一个参数的函数，必须是node-js风格      |
| func      | function  | 可选      | 从回调函数上接收值，然后将它们映射成一个新的值给输出流使用 |
| scheduler | Scheduler | 可选      | 使用scheduler控制何时调用回调函数                          |

返回值

function 返回一个 Observable的函数。

示例

    import * as fs from 'fs';

    var readFileAsObservable = Rx.Observable.bindNodeCallback(fs.readFile);

    var result = readFileAsObservable('./roadNames.txt', 'utf8');

    result.subscribe(x => console.log(x), e => console.error(e));

----

    someFunction((err, a, b) => {
        console.log(err); // null
        console.log(a); // 5
        console.log(b); // "some string"
    });

    var boundSomeFunction = Rx.Observable.bindNodeCallback(someFunction);

    boundSomeFunction()
    .subscribe(value => {
        console.log(value); // [5, "some string"]
    });

----

    someFunction((err, a, b) => {
        console.log(err); // undefined
        console.log(a); // "abc"
        console.log(b); // "DEF"
    });

    var boundSomeFunction = Rx.Observable.bindNodeCallback(someFunction, (a, b) => a + b);

    boundSomeFunction()
    .subscribe(value => {
        console.log(value); // "abcDEF"
    });

----

    someFunction(a => {
        console.log(a); // 5
    });

    var boundSomeFunction = Rx.Observable.bindNodeCallback(someFunction);
    boundSomeFunction()
    .subscribe(
        value => {}             // never gets called
        err => console.log(err) // 5
    );

## create

----
静态方法

创建一条流，当有观察者订阅时，执行传入的函数。

* <font face="仿宋">_创建一条自定义的流，可以在这条流上输出任何你想要的值。_</font>

                create(observer => observer.next(1));

    -1-------------------------------------------------->

这个操作符将一个onSubscription函数转换成一条流，当有订阅者订阅时，这个函数将使用一个观察者——observer作为唯一的参数调用，在函数执行的过程中，如果需要传递数据，则函数会通过调用观察者的相应方法将数据能过相应的通知传递给观察者。

调用next方法时，作为正常数据传递；调用complete方法时，会给观察都发送一个结束通知，然后流会关闭并且再也不会有任何通知发出。调用error方法时，可以传递错误发生的信息以使观察者知道流上发生错误的原因。

可以多次调用观察者上next方法向它传递值，但是 complete 和error 方法只能调用一次，并且在调用了它们中的任意一个时就不能再调用任何方法。如果在调用了 complete 或 error 方法后还执行了其它方法，根据流的特性原则，这些方法都会被忽略。需要注意的量并不一定要调用 complete 方法来结束流，你完全可以不调用它，从而创建一条永远不会结束的流。

onSubscription函数可以选择返回一个函数或者一个具有 unsubscribe 方法的对象，当需要取消订阅，释放流执行时的资源时，就可以调用这个返回的函数或者对象的 unsubscribe 方法。例如，假设你在流执行进程中使用了一个定时器，当取消订阅时就可以通过上述的方式将定时器清理掉以避免浪费计算资源。

大多数情况下并不会使用create方法去创建一条流，因为现有操作符可以为我们创建适合各种场景的流，换句话讲，这个操作符是比较底层的，如果现有的操作符实在无法满足需要，你可以考虑使用它。

typescript 签名问题

因为 Observable 继承的类已经定义了静态 create 方法,但是签名不同, 不可能给 Observable.create 合适的签名。 正因为如此，给 create 传递的函数将不会进行类型检查，除非你明确指定了特定的签名。

当使用 TypeScript 时，我们建议将传递给 create 的函数签名声明为(observer: Observer) => TeardownLogic, 其中Observer 和 TeardownLogic 是库提供的接口。

参数

| Name           | Type                                        | Attribute | Description                                                                                      |
| -------------- | :-----------------------------------------: | :-------: | -----------------------------------------------------------------------------------------------: |
| onSubscription | function(observer: Observer): TearDownLogic |           | 此函数在发生订阅时执行，在适当的时候通过相应的方法给观察者发送通知，执行后会返回清理资源的方法。 |

返回值

Observable 流。当流被订阅时会执行传入的函数传递数据。

示例

    Rx.Observable.create(function(observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.complete();
    })
    .subscribe(
        v => console.log(v),
        err => {},
        _ => console.log('observable completed')
    );

----
    Rx.Observable.create(function(observer) {
        observer.error('error occur');
    })
    .subscribe(
        v => console.log(v), // 永远不会被调用
        err => console.log(err),
        complete => console.log('completed') // 永远不会被调用
    )

## defer

----
静态方法

通过工厂函数创建一个新流的作为输出流，每当有新的订阅者要求订阅时，都会使用此函数创建一个新的流。

* <font face="仿宋">_创建一个新的内部流作为输出流。_</font>

                defer(() => Rx.Observable.of('a','b','c'));

    ------a-------b------c--|-->

当有新的订阅者订阅流时，通过工厂方法创建一个新的输出流给订阅者，工厂函数只会在有订阅者订阅时才执行。尽管看起来所有的订阅者订阅的都是同一条流，但事实下每个订阅者的都订阅了不同的输入流。

参数

| Name              | Type                                | Attribute | Description                                                                                                           |
| ----------------- | :---------------------------------: | :-------: | --------------------------------------------------------------------------------------------------------------------: |
| observableFactory | function(): Subscribable Or Promise |           | 工厂函数，可以用来生成新的流，每次订阅都会触发它的执行。可以返回一个promise，但是promise会立即被包装成一个 Observable |

返回值

Observable 对这个流的订阅会触发工厂函数的执行产生新的流。

示例

    Rx.Observable.defer(() => {
        if(Math.random() > 0.5) {
            return Rx.Observable.fromEvent(document, 'click');
        } else {
            return Rx.Observable.interval(1000);
        }
    });

## empty

----
静态方法

创建一个不会发出任何有效值的流，只会立即发出一个Complete通知。

* <font fact="仿宋">_立即发出一个结束通知。_</font>

            empty

    -|------------------------------->

这个静态操作符的唯一用处就是创建一个立即发出Complete通知的流。可以用来和其它操作符进行组合，比如：mergeMap。

参数

| Name      | Type      | Attribute | Description                       |
| --------- | :-------: | :-------: | --------------------------------: |
| scheduler | Scheduler | 可选      | 使用scheduler控制何时发出结束通知 |

示例

    var observable = Rx.Observable.empty().startWith(7);

    observable.subscribe(x => console.log(x));
----
    var interval = Rx.Observable.interval(1000);

    var result = interval.mergeMap(x => x % 2 === 1? Rx.Observable.of('a', 'b', 'c') : Rx.Observable.empty());

    result.subscribe(v => console.log(v));

## from

----
静态方法

将数组、类数组对象、promise、部署了遍历器接口的对象或类 Observable 对象转换成Observable。

* <font face="仿宋">_几乎可以将任何东西都转换成流。_</font>

                from([1,2,3])

    1--------------2--------------3|

可以将多种类型的数据转换成流，并且将原数据上的值依次推送到流上。字符串被当成由字母组成的数组进行转换。

参数

| Name      | Type               | Attribute | Description                                                                    |
| --------- | :----------------: | :-------: | -----------------------------------------------------------------------------: |
| ish       | ObservableInput<T> |           | 想要转换的数组、类数组对象、promise、部署了遍历器接口的对象或类Observable对象 |
| scheduler | Scheduler          | 可选      | 调节值的发送                                                                   |

示例

    var observable = Rx.Observable.from([1,2,3,4,5]);

    observable.subscribe(v => console.log(v));
----
    function* generatorDoubles(seed) {
        var i = seed;

        while(true) {
            yield i;

            i = i*2
        }
    }

    var iterator = generatorDoubles(3);
    var result = Rx.Observable.from(iterator).take(5);

    result.subscribe(v => console.log(v));

## fromEvent

----
静态方法

创建一个从指定的目标对象上发出指定的事件的流。

* <font face="仿宋">_基于DOM事件或者Node-js的事件发射器创建一条流。_</font>

                fromEvent(document, 'click');

    ---------------------------event--------event----------------->

此操作符接受一个事件对象作为第一个参数，在这个事件对象上注册了对应事件的处理函数，第二个参数是一个字符串，它表明需要监听哪种类型的事件。后边会介绍这个操作符支持的事件对象，如果需要监听其它的事件对象，需要使用fromEventPattern，它可以在任意的事件对象上使用。当处理支持fromEvent指定的API时，添加事件和移除事件的方法可能具有不同的名称，但它们都接接受一个描述事件类型的字符串和一个执行函数，执行函数在事件发生时会被调用。

一旦输出流被订阅，事件处理函数就会注册到目标对象上的相应事件，当事件被触发时，事件对象就会被当作第一个参数传递给输出流。当取消订阅时，事件处理函数就会从目标对象上移除。

注意，当目标对象调用事件处理函数时传入了多个参数，除了第一个参数外，其它参数都不会出现在输出流中。为了访问其它参数，可以传入一个可选的映射函数给fromEvent，这个映射函数可以接收到所有的参数，输出流上会输出映射函数返回的值。

以下列出的事件对象通过鸭式辩型进行检查。这意味着它并不关心传的是什么对象及及运行在哪种环境中，只要它暴露了相应的方法，都可以安全的使用fromEvent。举例来说就是假如在node环境中某一个对象拥有和DOM对象相同的方法时，依然可以使用fromEvent操作符。

支持的事件对象

    - DOM EventTarget 一个部署了addEventListener 和 removeEventListener方法的对象。在浏览器中，addEventListener接受一个描述事件类型的字符串，和一个事件处理函数，以及第三个可选的参数（一个对象或一个布尔值）用来描述事件处理函数何时被触发。在使用fromEvent时依然可能提供第三个可选参数。

    - Node.js EventEmitter 一个部署了addListener 和 removeListener方法的对象。

    - jQuery-style event target 一个包含on 和off方法对象。

    - DOM NodeList DOM节点列表，例如 document.querySelectorAll 或者 Node.childNodes 的返回结果。虽然这些集合并不是事件对象，fromEvent仍然可以通过遍历集合来给每一个节点注册事件。当取消订阅时，所有注册的函数也将被移除。

    - DOM HTMLCollection 和NodeList一样，也会给每一个DOM节点注册事件，并且在取消订阅时移除它们。

参数

| Name      | Type                       | Attribute | Description                                                                                                         |
| --------- | :------------------------: | :-------: | ------------------------------------------------------------------------------------------------------------------: |
| target    | EventTargetLike            |           | DOM EventTarget, Node.js EventEmitter, JQuery-like event target, NodeList 或者 HTMLCollection等可以添加事件的对象。 |
| eventName | string                     |           | 从目标对象上发出来的事件                                                                                            |
| options   | EventListenerOptions       | 可选      | 传入addEventListener的配置参数                                                                                      |
| selector  | SelectorMethodSingature<T> | 可选      | 结果的预处理函数，接受从事件处理函数上发出的所有参数，返回一个值                                                    |

返回值

    Observable<T>

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');

    clicks.subscribe(v => console.log(v));

----

    var clicksInDocument = Rx.Observable.fromEvent(document, 'click', true);

    var clicksInDiv = Rx.Observable.fromEvent(someDivInDocument, 'click');

    clicksInDocument.subscribe(_ => console.log('document'));

    clicksInDiv.subscribe(_ => console.log('div'));

## fromEventPattern

----
静态方法

从一个基于 addHandler/removeHandler 方法的API创建 Observable。

* <font face="仿宋">_将任何部署 addHandler/removeHandler 方法的API转化为 Observable。_</font>

创建一个流，通过 addHandler 和 removeHandler 方法来添加和移除事件处理程序，可以使用可选的映射函数来预处理结果。订阅时调用 addHandler 方法，取消订阅时调用 removeHandler 方法。

| Name          | Type                                            | Attribute | Description                                                                |
| ------------- | :---------------------------------------------: | :-------: | -------------------------------------------------------------------------: |
| addHandler    | function(handler: function): any                |           | 使用一个事件处理函数作为参数的函数，将处理函数添加到事件源                 |
| removeHandler | function(handler: function, singal?: any): void |           | 可选的函数，接受处理器函数作为参数，清理之前在事件源上注册的处理函数       |
| selector      | function(...args, any): T                       | 可选      | 输出值的预处理函数，接受从事件处理函数上输入的参数，返回的值在输出流的输出 |

返回值

    Observable

示例

    function addClickHandler(handler) {
        document.addEventListener('click', handler);
    }

    fuction removeClickHandler(handler) {
        document.removeEventListener('click', handler);
    }

    Rx.observable.fromEventPattern(
        addClickHandler,
        removeClickHandler
    )
    .subscribe(v => console.log(v));

## fromPromise

----
静态方法

将promise转换成流。

* <font face="仿宋">_将promise转换成流，随后将promise成功 resolve 后发出的值推送到流上，然后发出结束通知。_</font>

    ----------------------resolvedValue->

                fromPromise(promise);

    ---------------------resolvedValue|

将ES2015的promise或者符合 promise A+ 规范的promise转换成流。promise成功**resolve**后的值将会被当作正常值通过next方法推送到输出的流上，**reject**后的异常将会通过error方法推送到输出流上。

| Name      | Type           | Attribute | Description                             |
| --------- | :------------: | :-------: | --------------------------------------: |
| promise   | PromiseLike<T> |           | 需要转换的promise                       |
| scheduler | Scheduler      | 可选      | 调节值（来自 resolve 或 reject ）的发送 |

示例

    var result = Rx.Observable.fromPromise(fetch('http://myserver.com'));

    result.subscribe(v => console.log(v), err => console.error(err));
----
    var promise = new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('success');
        },2000);
    });

    Rx.Observable.fromPromise(promise).subscribe(v => console.log(v));

## interval

----
静态方法

创建一个按一定时间间隔发送整数的流。

* <font face="仿宋">_按时间间隔发送递增数字。_</font>

                interval(1000);

    --0--1--2--3--4--5--6--7--8--9--10------------->

这个操作符生成一个发出无限数列的流，传入的参数将作为发送的时间间隔，以毫秒数为单位。第一个值并不会立即发出，而是需要等到指定的时间间隔之后才发出。默认情况下，这个操作符使用async调度器提供时间概念，当然你可选传入其它的调度器来提供时间概念。

| Name      | Type      | Attribute       | Description                          |
| --------- | :-------: | :-------------: | -----------------------------------: |
| period    | number    | 可选。默认值：0 | 发送值的时间间隔，以毫秒数为单位     |
| scheduler | Scheduler | 可选。默认值：0 | 调节值的发送，为值的发送提供时间概念 |

返回值

Observable 每隔一段时间就发出一个整数的流。

示例

    Rx.Observable.interval(1000)
        .subscribe(v => console.log(v));

## never

----
静态方法

创建一个永远不会发出任何值的流。

* <font face="仿宋">_一条不会发出任何值的流。_</font>

                never

    ----------------------------------------->

这个操作符用来创建一个即不会发出正常通知，也不会发出错误通知和完成通知的流。可以用于测试或者和其它流进行合并。需要注意的是它不会发出完成通知，仍然需要手动取消订阅。

返回值

    Observable 不会发出任何值的流。

示例

    function info() {
        console.log('Will not be called');
    }

    Rx.Observable.never().startWith(7)
        .sbuscribe(v => console.log(v), info, info);

## of

----
创建一个流，把传入此函数的参数从左到右依次推送到流上，然后发出结束通知。

* <font face="仿宋">_发送传入的参数，然后发出结束通知。_</font>

            of(1,2,3);

    1-------------2--------------3|

用来创建一个简单的发送传入参数的流，这样可以用来和其它的流进行组合。默认情况下，它使用null作为调度器，也就是说会同步的把值发送出去，当然你可以自定义的使用其它的调度器来决定如何发送值。

参数

| Name      | Type      | Attribute | Description  |
| --------- | :-------: | :-------: | -----------: |
| values    | ...T      |           | 参数         |
| scheduler | Scheduler | 可选      | 调节值的发送 |

示例

    var numbers = Rx.Observable.of(10,20,30);

    var letters = Rx.Observable.of('a','b','c');

    var interval = Rx.Observable.interval(1000);

    var result = number.concat(letters).concat(interval);

    result.subscribe(v => console.log(v));

## repeat

----
实例方法

将已经发射过的值重复发射，直到达到指定的次数。

    -----a------b---|

        repeat(3);

    ----a-------b----a-------b----a-------b|

参数

| Name  | Type   | Attribute | Description                         |
| ----- | :----: | :-------: | ----------------------------------: |
| count | number | 可选      | 需要重复的次数，0表示发出一个空的流 |

## repeatWhen

----
实例方法

返回一个输入流的镜像，除了结束通知。如果输入流发出了完成通知，这个操作符会发出通知给通知流。如果通知流上发生了错误或者发出了完成通知，那么它会在子订阅上发送错误或完成通知，否则这个操作符会重新订阅输入流。

    ------------------r--------------r---------r-|--->

    --a---b------|-->

                repeatWhen

    --a---b----------a---b-----------a---b-----a-|-->

参数

| Name    | Type                                            | Attribute | Description                                                        |
| ------- | :---------------------------------------------: | :-------: | -----------------------------------------------------------------: |
| notifer | function(notifications: Observable): Observable |           | 接收一个发出通知的流，通知流上可以通过发错误或结束通知结束重复行为 |

返回值

    Observable 把输入流使用重复逻辑包装过后流。

示例

    const source = Rx.Observable.interval(1000).take(5);

    source
        .repeatWhen(() => Rx.Observable.fromEvent(document, 'click'))
        .subscribe(v => console.log(v));

----

    Rx.Oservable.fo(42)
        .repeatWhen(notifications => notifications // 输入流的上完成通知组成的流。
            .scan((acc, cur, i) => acc + i, 0)
            .delay(200)
            .takeWhile(v => v < 1)
        )
        .subscribe(v => console.log(v), e => console.error(e), () => console.log('complete'));

## range

----
静态方法

创建一个按顺序发出指定范围内的数字的流。

* <font face="仿宋">_按顺序发出指定范围内的数字。_</font>

                range(1,10);

    ---1---2---3---4---5---6---7---8---9---10-|->

range操作符会按顺序发出一定范围内的整数，接受一个起始值作为第一个参数，一个长度值作为第二个参数。默认情况下不使用任何调度器，只是将这些数字同步的发出，也可以使用一个调度器来决定如何发送值。

参数

| Name      | Type      | Attribute       | Description    |
| --------- | :-------: | :-------------: | -------------: |
| start     | number    | 可选。默认值：0 | 起始值         |
| count     | number    | 可选。默认值：0 | 需要发送的数量 |
| scheduler | Scheduler | 可选            | 调节值的发送   |

返回值

Observable 发送一个有限范围内的整数的流。

示例

    Rx.Observable.range(1,10)
        .subscribe(v => console.log(v));

## throw

----
静态方法

创建一个立即发出错误通知的流。

                throw(e);

    X------------------------>

要以用来和其它流进行合并，比如mergeMap。

参数

| Name      | Type      | Attribute | Description      |
| --------- | :-------: | :-------: | ---------------: |
| error     | any       |           | 错误通知发出的值 |
| scheduler | Scheduler | 可选      | 调节值的发送     |

返回值

    Observable 只发出错误通知的流。

示例

    Rx.Observable.throw(new Error('oops!)).startWith(7)
        .subscribe(v => console.log(v), e => console.error(e));

----

    Rx.Oservable.interval(1000)
        .mergemap(x => x === 13 ? Rx.Observable.throw('Thirteens are bad') : Rx.Observable.of('a', 'b', 'c'))
        .subscribe(v => console.log(v), e => console.error(e));

## timer

----
静态方法

创建一个输出流，在指定的延迟时间到达后开始发射值，在指定的间隔时间到达后发射递增过的值。

* <font face="仿宋">_类似于interval，但是这个操作符允许指定流开始发射值的时间。_</font>

                timer(3000, 1000);

    ------0--1--2--3--4--5--------->

timer在等待时间到达后开始发射值，开始发射后以指定的间隔时间发出递增的值，这些值是一些数字常量。等待时间可以是一个毫秒数，也可以是一个日期对象。默认情况下，使用async调度器提供时间的概念，但是允许传递任何类型的调度器，如果没有指定时间周期，输出流上只会发射0，反之，它会发出一个无限的数列。

| Name         | Type      | Attribute | Description                              |
| ------------ | :-------: | :-------: | ---------------------------------------: |
| initialDelay | number    | Date      |                                          |  |
| period       | number    | 可选      | 输出流上发出值的间隔时间                 |
| scheduler    | Scheduler | 可选      | 用来提供时间概念时调度器，会影响值的发送 |

返回值

Observable 等待时间到达后发出第一个值0，然后以一定的间隔时间发送递增的数字。

示例

    Rx.Observable.timer(3000, 1000)
        .subscribe(v => console.log(v));

----

    Rx.Observable.timer(5000)
        .subscribe(v => console.log(v));