# 组合类型操作符

## combineAll

----
静态方法

将高阶流转换成一阶流。当输入流完成后，内部使用combineLatest把内部流转化成输出流。

* <font face="仿宋">_使用combineLatest将高阶流打平成低阶流。_</font>

    -------------------------------------|-------------------------------->
        \          \---1---2--|->         \---1---2--|->
         \--a--b-|->                      \--a--b--|->

                combineAll

    -------------------------------------|-a1-b1-b2-|->

从一个高阶流上收集内部流，当高阶流完成后，开始订阅所有的内部流，使用combineLatest的方式把所有值合并到输出流上。

    - 每当内部流发出值时，输出流都会发出值。
    - 输出流上发出的值取决于是否提供了聚合函数：
        - 提供聚合函数时：聚合函数按次序接收每个内部流上的最新值，把运行的结果传递给输出流。
        - 未提供聚合函数时：把各个内部流的最新值按次序以数组的形式传递给输出流。

| Name    | Type     | Attribute | Description                                                                  |
| ------- | :------: | :-------: | ---------------------------------------------------------------------------: |
| project | function | 可选      | 聚合函数，可以按次序接收各个内部流的最新值，函数运行后的结果将传递给输出流。 |

返回值

Observable 输出各内部流的值聚合后的结果的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .map(event => Rx.Observable.interval(Math.random()*200).take(3))
        .take(2)
        .combineAll()
        .subscribe(x => console.log(x));

## combineLatest

----
实例方法

将多个输入流组合成一个输出流，输出流上的数据是由每一条输入流上的最后一条数据组合成的数据。

* <font face="仿宋">_无论什么时间只要输入流上发出了数据，它就会把所有输入流上的最新数据组合成一条新的数据推送到输出流上。_</font>

    -a------b----c-----d------e---|

    --1--2-----3--4-----5---6---|

            combineLatest

    --a1-a2-b2--b3-c3-c4----d4-d5---e5-e6-|

combineLatest把源 Observable 上的值和作为参数传递进来的 Observable 的值组合了起来。它的实现方式是通过订阅每一条Observable，将它们发出的最新值以数组的方式收集起来再推送到输出流上，当然也可以先传递给一个映射函数，由映射函数先把值处理后再推送到输出流上。

参数

| Name    | Type            | Attribute | Description                                |
| ------- | :-------------: | :-------: | -----------------------------------------: |
| other   | ObservableInput |           | 用来进行组合的输入流，可以输入多条输入流   |
| project | function        | 可选      | 用于将组合后的值映射成一个新值发送给输出流 |

返回

Observable 一条包含所有输入流最新值的流，或者是所有最新值的映射值的流。

示例

    var weight = Rx.Observable.of(70,72,76,79,75);

    var height = Rx.Observable.of(1.76,1.77,1.78);

    var bmi = weight.combineLatest(height, (w, h) => w/h*h);

    bmi.subscribe(x => console.log('BMI is ' + x);

## concat

----
实例方法

在当前的流发出值后，按照输入流的顺序再依次发出各输入流上发出的值。

* <font face="仿宋">_将多个流按顺序连接起来，依次发出输入流上发出的值。_</font>

    ----a-------b--|-->

                    -------c-----d----|>

                concat

    ----a-------b----------c-----d----|>

将源 Observable 和 其它的 Observable 连接起来。从源 Observable 开始逐个订阅输入流，并将获取到的值传递给输出流。在开始订阅下一个输入流之前必须保证当前的流已经发出了结束通知。

参数

| Name      | Type            | Attribute        | Description              |
| --------- | :-------------: | :--------------: | -----------------------: |
| other     | ObservableInput |                  | 将要被连接的流           |
| scheduler | Scheduler       | 可选。默认：null | 调节对每一条输入流的订阅 |

返回值

Observable 按顺序排列的所有输入流的值构成的流。

示例

    var timer = Rx.Observable.interval(1000).take(4);
    var sequence = Rx.Observable.range(1,10);
    var result = time.concat(sequence);

    result.subscribe(v => console.log(v));
----
    var timer1 = Rx.Observable.interval(1000).take(10);
    var timer2 = Rx.Observable.interval(2000).take(6);
    var timer3 = Rx.Observable.interval(500).take(10);
    var result = timer1.concat(timer2, timer3);

    result.subscribe(x => console.log(x));

## concatAll

----
实例方法

通过把内部流串联起来的方式把高阶流打平成一阶流。

* <font face="仿宋">_把内部流按次序连接起来作为输出流。_</font>

    -----------------------------------|----------------------------|-->
         \a----b---|->   \-1--2--|->

                    concatAll

    -----------------------------------a----b---------1--2--|->

连接所有从输入流上发出的内部流。这个操作符会依次监听每一个内部流，也就是说只有当一个内部流发出完成通知后它才会继而监听下一个，然后将监听得到的所有值依次发送给输出流。

注意：如果输入流迅速且无止境的发出内部流，而内部流发出结束通知的时间比输入流慢，那么在不设置缓冲区时可能会面临内存问题。

concatAll 和 mergeAll使用并发参数为1时的行为是相同的。

返回值

Observable 依次发出所有内部流上的值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .map(event => Rx.Observable.interval(1000).take(4))
        .concatAll()
        .subscribe(v => console.log(v));

## exhaust

----
实例方法

将高阶流打平成一阶流，如果在一个内部流完成之前又有一个新内部流开始发射值，那么新的内部流将被忽略。

* <font face="仿宋">_当前的内部流正在发射值时忽略掉新产生的内部流。_</font>

    -------------------------------------------------|-->
            \      \                  \--1---2--3--4-|->
             \      \--e---f---g---h--i-|->
              \---a----b----c----d-|->

                exhaust

    --------a----b----c----d-------------1---2--3--4-|->

此操作符将会订阅一个高阶流，当高阶流发出内部流时输出流开始订阅内部流，如果在订阅一个内部流期间又一个新的内部流到达而正在订阅的内部流又没有结束，这个新流将会被忽略。只有当前订阅的内部流完成后才会开始订阅下一个内部流。简而言之就是只有当前流的值都耗尽后才可能执行新的订阅。

返回值

Observable 输出流的上必然输出第一条内部流的值，其它的内部流的值是否会在输出流上取决于上述的规则。

示例

    Rx.Observable.fromEvent(document,'click')
        .map(event => Rx.Observable.interval(1000).take(5))
        .exhaust()
        .subscribe(v => console.log(v));

## forkJoin

----
静态方法

将输入流的最后一个值合并后传给输出流。

* <font face="仿宋">_等待输入流结束，然后将最后发出的值传给输出流。_</font>

forkJoin可以以参数或数组的形式接收任意数量的输入流。如果没有转入输入流，输出流将会立即发出结束通知。

它会等所有的输入流发出结束通知，然后发出一个包含所有输入流发出的最终值组成的数组，因此，传入n条输入流，得到的数组中将包含n个元素，每一个元素都是从对应顺序的输入流上取到的最后一个值。这也就意味着输出流只会发出一个值，紧接着就会发出结束通知。

为了使用输出流上获取到的数组的长度与输入流的数量保持一致，如果某一个输入流在发出结束通知之前没有发出过任何有效值，那么输出流也将会立刻结束并且不再发出任何值，尽管其它的输入流上可能会发出有效值。反过来，假如有一条输入流一直没有发出结束通知，输出流也不会发出值，除非另外一条输入像之前描述的那样只发了结束通知。总的来说就是为了让输出流发出值，所有的输入流都必须发出结束通知而且在此之前必须至少发出一个有效值。

如果输入流中的某一条发出了错误通知，输出流将会立即发出这个错误通知，并且立即取消对其它流的订阅。

此外，forkJoin还可以接受一个可选的映射函数，它会接受从各输入流上得到的值作为参数，然后把这些值映射成另外一个值发送给输出流。也就是说你可以认为默认的映射函数是把这些值放入一个数组里交给了输出流。注意这个函数只有在输出流准备发出值的时候才会被调用一次。

    ----2---3------4----5--|-->

    ----a----b------------d---|-->

                forkJoin

    --------------------------5d|

参数

| Name    | Type                     | Attribute | Description                                              |
| ------- | :----------------------: | :-------: | -------------------------------------------------------: |
| sources | ...SubscribableOrPromise |           | 任意数量的输入流，可以直接以参数的形式或者放在数组中传入 |
| project | function                 | 可选      | 接受所有输入流产生的值作为参数，输出一个值给输出流       |

返回值

Observable 以数组形式获取到的每一个输入流的值，或者来自映射函数的值。

示例

    const observable = Rx.Observable.forkJoin(
        Rx.Observable.of(1, 2, 3, 4),
        Rx.Observable.of(5, 6, 7, 8)
    );

    observable.subscribe(
        value => console.log(value),
        err => {},
        () => console.log('This is how it ends!')
    );
----
    const observable = Rx.Observable.forkJoin(
        Rx.Observable.interval(1000).take(3),
        Rx.Observable.interval(500).take(4)
    );

    observable.subscribe(
        value => console.log(value),
        err => {},
        () => console.log('This is how it ends!')
    );
----
    const observable = Rx.Observable.forkJoin(
        Rx.Observable.interval(1000).take(3), // emit 0, 1, 2 every second and complete
        Rx.Observable.interval(500).take(4), // emit 0, 1, 2, 3 every half a second and complete
        (n, m) => n + m
    );

    observable.subscribe(
        value => console.log(value),
        err => {},
        () => console.log('This is how it ends!')
    );

## merge

----
实例方法

创建一条可以同时把所有输入流的值推送出去的流。

* <font face="仿宋">_把多条输入流打平成一条输入流，所有输入流的值都可以在这条输出流上得到。_</font>

    -a----------b----------c---|----->

    ------d---------e----------e----|-->

                merge

    -a----d----b----e-----c----e----|-->

合并所有输入流的订阅，不管是源 Observable 还是作为参数输入的 Observable，只是把值依次推送到输出流上，不作任何额外的更改。输出流只有在所有的输入流都完成以后才能完成，任何一条输入流上的错误都将立即推送到输出流上。

参数

| Name       | Type            | Attribute                              | Description                                  |
| ---------- | :-------------: | :------------------------------------: | -------------------------------------------: |
| other      | ObservableInput |                                        | 用来进行组合的流，可以传递多个输入流作为参数 |
| concurrent | number          | 可选。默认值：Number.POSITIVE_INFINITY | 允许同时订阅的输入流的最大数量               |
| scheduler  | Scheduler       | 可选。默认值：null                     | 用来调节输入流的并发                         |

返回值

Observable 可以发送所有输入流上的值的Observable。

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var timer = Rx.Observable.interval(1000);
    var clicksOrTimer = clicks.merge(timer);

    clicksOrTimer.subscribe(x => console.log(x)); ---- var timer1 = Rx.Observable.interval(1000).take(10); var timer2 = Rx.Observable.interval(2000).take(6);
    var timer3 = Rx.Observable.interval(500).take(10);
    var concurrent = 2;

    // 这里只允许同时订阅2条，所以它的输出流的行为是先发出timer1和timer2上的值，
    // 10秒过后当timer1发出完成通知后，开始同时发射timer2和timer3的值，所有输入流
    // 的值都发射完成后再发出完成通知。
    var merged = timer1.merge(timer2,timer3,concurrent);

    merged.subscribe(v => console.log(v));

## mergeAll

----
实例方法

将高阶流打平成一阶流，并行发送所有内部流的值。

    -------------------------------------|---->
        \               \-1-2--3-4--5--6-|->
         \-a--b--c--d--e--f--g--h--i-|->

            mergeAll

    ------a--b--c--d--e--f-1-2-g-3-j-4-i-5--6-|->

此操作符订阅所有的内部流，并把所有内部流上发射的值传送给输出流。只有当所有的内部流都完成时输出流才会发出完成通知。如果有一个内部流发出错误通知，这个错误会立即被发送给输出流。

参数

| Name       | Type   | Attribute                            | Description              |
| ---------- | :----: | :----------------------------------: | -----------------------: |
| concurrent | number | 可选。默认：Number.POSITIVE_INFINITY | 并发订阅内部流的最大数量 |

返回值

Observable 并行输出所有内部流的值的流

示例

    Rx.Observable.fromEvent(document,'click')
        .map(event => Rx.Observable.intervl(1000))
        .mergeAll()
        .subscribe(v => console.log(v));

    Rx.Observable.fromEvent(document, 'click')
        .map(event => RxObservable.interval(1000).take(10))
        .mergeAll(2)
        .subscribe(v => console.log(v));

## race

----
实例方法

把最先开始输出值的流作为输出流。

    --------a-------b-----c------|-->

    ---1------3------|-->

                race

    ---1------3------|-->

参数

| Name           | Type           | Attribute | Description      |
| -------------- | :------------: | :-------: | ---------------: |
| ...observables | ...observables |           | 参与竞赛的输入流 |

返回值

Observable 第一条发出值的输入流的镜像。

示例

    const first = Rx.Observable.of('first')
        .delay(100)
        .map(_ => throw 'error');

    const second = Rx.Observable.of('second').delay(200);

    const third = Rx.Observable.of('third').delay(300);

    Rx.Observable.race(first, second, third)
        .subscribe(v => console.log(v));

    ----

    Rx.Observable.interval(2000)
        .race(
            Rx.Observable.interval(1000),
            Rx.Observable.fromEvent(document, 'click')
        )
        .subscribe(v => console.log(v));

## startWith

----
实例方法

在发送流上的数据时，先将传入此操作符的参数作为流上的第一个值发送出去。

    -------a--------b-------c--|-->

                startWith(s)

    s------a--------b-------c--|-->

参数

| Name      | Type      | Attribute | Description                |
| --------- | :-------: | :-------: | -------------------------: |
| value     | ...T      |           | 输出流上发出的第一个值     |
| scheduler | Scheduler | 可选。    | 用来调节Next通知上发出的值 |

返回值

Observable 先发出指定的参数，然后再发出输入流上的值。

示例

    var source = Rx.Observable.interval(1000);

    source.startWith('start')
        .subscribe(v => console.log(v));

## switch

----
实例方法

通过取消之前的订阅，重新订阅最新的内部流来把高阶流打平成一阶的流。

* <font face="仿宋">_打平高阶流，当有新的流到达时，立即放弃对之前的流的订阅。_</font>

    -------------------------------|-------->
          \          \----e----f---|>
           \---a---b-----c---d---|->

                switch

    -----------a---b-----e----f---|

switch用来订阅一个高阶流（发出流的流）。一个内部流被源 Observable 发出后，它就会被输入流订阅，并且将条内部流上的值传递到输出流上，比时它的行为和mergeAll是一样的，但是当下一个新的内部流被发出后，switch将会取消之前订阅的那条内部流，继而转向订阅这条新的内部流。

返回值

Observable 从源 Observable 上取到的最新的内部流。

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var higherOrder = clicks.map(event => Rx.Observable.interval(1000));
    var switched = higherOrder.switch();

    switched.subscribe(v => console.log(v));

## withLatestFrom

----
实例方法

将多个输入流组合成一个输出流，输出流上的数据是由每一条输入流上的最后一条数据组合成的，但是组合动作只在源 Observable 发出值时发生。

* <font face="仿宋">_当源 Observable 上发出值时，它会从所有的输入流上获取值，将它们组合成一个新的值推送给输出流。_</font>

    -a-----b--------------c------------d--e---|-->

    ----1-----2-----3--4--------5----|---->

            withLatestFrom

    ------b1------------c4-----------d5--e5---|-->

combineLatest将源 Observable 上的值和以参数形式传入的 Observable 上的值组合起来发送给输出流，只有当源 Observable 上发出值时才进行组合。还可以传入一个映射函数处理组合后的值。需要注意的是，只有当所有的输入流都发出过值以后输出流上才会产生值。

参数

| Name    | Type            | Attribute | Description                                                                        |
| ------- | :-------------: | :-------: | ---------------------------------------------------------------------------------: |
| other   | ObservableInput |           | 用来进行组合的输入流，可以输入多条输入流                                           |
| project | function        | 可选      | 用于将组合后的值映射成一个新值发送给输出流。所有输入流的值依次作为参数传入此函数。 |

返回值

Observable 一条包含所有输入流最新值的流，或者是所有最新值的映射值的流。

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var timer = Rx.Observable.interval(1000);
    var result = clicks.withLatestFrom(timer)

    result.subscribe(x => console.log(x));

## zip

----
实例方法

如果所有的输入都是流，那么将所有输入流对应位置上的值组合成一个新值，将这个新值作为输出流上的正常值。此外zip操作符还可以直接使用可转化为流的数据作为参数，比如数组，promise，字符串等。

    --a------b------c------d---2--|->

    -----e------f------g-----h--|->

                zip

    ----ae------fb-----gc----hd-|->

返回值

Observable 将各输入流上对应的值组合成新的值发送给输出流或者先经过映射函数处理后再发送给输出流。

参数

| Name        | Type     | Attribute | Description                                                       |
| ----------- | :------: | :-------: | ----------------------------------------------------------------: |
| observables | any      |           | 用来组合的流或者它可以转化成流的值，比如promise，数组，字符串等。 |
| project     | function | 可选      | 可选的映射函数，将组合后的值映射成新的值推送给输出流。            |

示例

    var obs1 = Rx.Observable.from([1,2,3,4]);
    var obs2 = Rx.Observable.from(['a','b','c']);

    obs1.zip(obs2)
        .subscribe(v => console.log(v));
----
    var obs1 = Rx.Observable.interval(1000);
    var obs2 = Rx.Observable.from(['a', 'b','c']);

    obs1.zip(obs2,(clock, letter) => letter + clock)
        .subscribe(v => console.log(v));
----
    var obs = Rx.Observable.interval(1000);
    var promise = new Promise(resolve => {
        setTimeout(() => resolve('hello'), 2000);
    });

    obs.zip(promise, (obs, promise) => promise + obs)
        .subscribe(v => console.log(v));

## zipAll

----
实例方法

参数

| Name        | Type     | Attribute | Description                                                       |
| ----------- | :------: | :-------: | ----------------------------------------------------------------: |
| project     | * |       |      |

返回值

Observable

示例

    const obs1 = Rx.Observable.interval(1000).take(5);

    const obs2 = Rx.Observable.interval(1000).mapTo('a').take(4);

    Rx.Observable.of(obs1,obs2)
        .zipAll()
        .subscribe(v => console.log(v));

    ----

    Rx.Observable.of(obs1, obs2)
        .zipAll((a, b) => a + b)
        .subscribe(v => console.log(v));