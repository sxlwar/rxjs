# 创建型操作符

## bindCallback

## bindNodeCallback

## create

## defer

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

## fromEventPattern

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

## generate

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