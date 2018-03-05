# 过滤类型操作符

## debounce

----
实例方法

只有在由延迟流决定的一段特定时间内，输入流上没有再发出新的值后，输入流发出的值才会被输出流发了。

* <font face="仿宋">_类似于debuounceTime，但它的静默时间由第二个流决定。_</font>

    -----a-------b--c------------d-----|-->

    ---------|-->

                debounce

    ---------a----------------c------------d--|-->

延迟输入流上值的发送，在静默阶段内如果有新的值到来那么前一个静默值会被丢弃。这个操作符会追踪输入流上的最新值，同时通过调用传入的函数来获取一条决定静默时间的流。从输入流上发出的值只有在延迟流发出值或完成通知时才会被输出流发出。如果在延迟流发出值之前有新的值从输入流上发出，那么前一个值就会被丢弃，不会出现在输出流上。

和 debounceTime 一样，这是一个限制频率的操作符，也是一个延迟发射值的操作符，输出流的值始终不会和输入流的值同步发出。

参数

| Name             | Type                                      | Attribute | Description                                               |
| ---------------- | :---------------------------------------: | :-------: | --------------------------------------------------------: |
| durationSelector | function(value: T): SubscribableOrPromise |           | 接受输入流上的值，用于计算延迟时间，返回一个流或者promise |

返回值

    Observable 延迟发送输入流的值，同时可以限制发送频率的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .debounce(() => Rx.Observable.interval(1000))
        .subscribe(v => console.log(v));

## debounceTime

----
实例方法

在指定的时间范围内，如果输入流上没有再发出值，输入流的上一值才会被发出。

* <font face="仿宋">_和delay的行为有些相似，但是在有大量的发送时，它只会发送最新值。_</font>

    ----a----b-c----d---------e-j--------|->

                delay(20);

    ----a------c----d-----------j--------|->

这个操作符会延迟发送输入流上传递来的值。在延迟时间内如果有新的值抵达，那么它会把之前值丢弃，以这个新值抵达的时间为基准重新计算延迟时间，只有在延迟时间过后没有新值抵达时才会发送这个值，否则它重复上述的行为。

这是一个可以限制发送频率的操作符，因为它限制了在一定的时间只能有一个值发出，它也类似一个延迟操作符，因为输出流上的值的发送时间与输入流的发送时间并不相同。这个操作符可以接收一个调度器。

参数

| Name      | Type      | Attribute           | Description                                                                        |
| --------- | :-------: | :-----------------: | ---------------------------------------------------------------------------------: |
| dueTime   | number    |                     | 以毫秒为单位的时间值（或者由同调度器提供的时间单位），用来决定发送值之前的静默时间 |
| scheduler | Scheduler | 可选。默认值：async | 用来调节操作符内部的时间                                                           |

返回值

Observable 根据指定的时间延迟发送输入流上的值，并且在发送的频率过高时会丢弃之前的值。

示例

    Rx.Observable.fromEvent(document, 'click')
        .debounceTime(1000)
        .subscribe(v => console.log(v));

## distinct

----
实例方法

创建一个输出流，它会将输入流上发射的当前值与前一个值进行对比，如果值发生了改变，这个值将会在输出流上推送。

如果提供一个key值选择函数作为参数，将使用这个函数把输入流上的值映射成一个新值后再执行对比，如果没有提供引函数，将直接使用输入流上的值进行相等检查。

在javascript中存在 set 数据结构，这个操作符会使用 set 结构来提高检查的性能。

运行时，此操作符依赖于数组及数组的indexOf方法最小化实现 set 结构，所以如果使用多个值进行等值检查时运行性能会降低。即使是在最新的浏览器中，一个运行时间过长的等值检查可能会引发内存泄露。为了使这种情况得到缓解，某些情况下可以经它提供第二个可选参数——flushes，这样内部的set结构的值就可以被清空。

参数

| Name        | Type       | Attribute | Description                     |
| ----------- | :--------: | :-------: | ------------------------------: |
| keySelector | function   | 可选      | 用来决定使用哪个值进行等值检查  |
| flushes     | Observable | 可选      | 用来刷新操作符内部的set结构的值 |

返回值

Observable 将源 Observable 上的值进行区分过的流。

示例

    Rx.Observable.of(1,1,2,2,3,3,4,4,5,7,5,7,8)
        .distinct()
        .subscribe(v => console.log(v));
----
    Rx.Observable.of({
        { age: 4, name: 'Foo' },
        { age: 7, name: 'Bar' },
        { age: 5, name: 'Foo' }
    })
    .distinct(v => v.name)
    .subscribe(v => console.log(v));
----
    Rx.Observable.of([1,2,3],[1,2,3],[0,1,2])
        .distinct(v => v)
        .subscribe(v => console.log(v));

## distinctUntilChanged

----
实例方法

创建一个输出流，它会将输入流上发射的当前值与前一个值进行对比，如果值发生了改变，这个值将会在输出流上推送。

如果提供一个比较函数，将使用它来比较当前值与前一个值以决定该值是否应该在输出流上发出。如果没有提供此函数，将使用默认的比较函数。默认的比较函数为：(x,y) => x === y;

| Name    | Type     | Attribute | Description                                |
| ------- | :------: | :-------: | -----------------------------------------: |
| compare | function | 可选      | 用来比较输入流上的当前值与前一个值是否相同 |

返回值

Observable 对输入流的值进行过区分后的流。

示例

    Rx.Observable.of(1,1,2,2,2,1,1,3,3,4)
        .distinctUntilChanged()
        .subscribe(v => console.log(v));
----
    Rx.Observable.of({
        { age: 4, name: 'Foo' },
        { age: 7, name: 'Bar' },
        { age: 5, name: 'Foo' }
    })
    .distinctUntilChanged((pre, cur) => cur.name === pre.name)
    .subscribe(v => console.log(v));

## distinctUntilKeyChanged

----
实例方法

创建输出流，使用提供的 key 获取源 Observable 值的 value，比较当前值与前一个值是否相等，只有发生了变化时才会在输出流上发出。

如果提供一个比较函数，则使用此函数来判定输入流发出的值是否应该在输出流上发出，如果没有提供此函数，默认使用等值检查。

参数

| Name    | Type     | Attribute | Description                                |
| ------- | :------: | :-------: | -----------------------------------------: |
| key     | string   |           | 进行比较的 value 的key名                   |
| compare | function | 可选      | 用来比较输入流上的当前值与前一个值是否相等 |

返回值

Observable 对输入流的值进行过区分后的流。

示例

    Rx.Observable.of(
            { age: 4, name: 'Foo'},
            { age: 7, name: 'Bar'},
            { age: 5, name: 'Foo'},
            { age: 6, name: 'Foo'}
        )
        .distinctUntilKeyChanged('name')
        .subscribe(x => console.log(x));
----
    Rx.Observable.of(
        { age: 4, name: 'Foo1'},
        { age: 7, name: 'Bar'},
        { age: 5, name: 'Foo2'},
        { age: 6, name: 'Foo3'}
        )
        .distinctUntilKeyChanged('name', (x, y) => x.substring(0, 3) === y.substring(0, 3))
        .subscribe(x => console.log(x));

## elementAt

----
实例方法

只输出输入流上指定位置的值然后立即结束的流。

    ---------a--------b---------c------d------|-->

                elementAt(2);

    ----------------------------c|->

输出流输出输入流上指定索引位置的值或者在索引超出输入流的范围后输出一个默认值。如果超出索引范围，但是又没有提供默认值，此时输出流上会输出一个错误通知。

参数

| Name         | Type   | Attribute | Description     |
| ------------ | :----: | :-------: | --------------: |
| index        | number |           | 从0开始的索引值 |
| defaultValue | T      | 可选      | 默认值          |

返回值

Observable 只输出一个值的流，可能是输入流的正常值，也可能是默认值，也可能是一个错误。

错误

ArgumentOutOfRangeError 如果传入的索引小于0，或者输入流在索引位置上没有值时这个错误会被抛出。

示例

    Rx.Observable.fromEvent(document, 'click')
        .elementAt(2)
        .subscribe(v => console.log(v));

## filter

----
实例方法

创建一个流，它的值是使用判定函数对输入流发出的值进行过滤后的值。

* <font face="仿宋">_与数组的filter方法类似，发出过滤后的值。_</font>

    --0--1--2--3--4--5----|-->

                filter(v => v % 2 === 0);

    --0-----2-----4-------|>

和数组的filter方法行为一样，从输入流上获取值，使用判定函数对值进行过滤，只有符合过滤条件的值才会在输出流上发出。

| Name      | Type                                   | Attribute | Description                                                                                                                                                    |
| --------- | :------------------------------------: | :-------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| predicate | function(v: T, index: number): boolean |           | 判定函数。它从输入流上接收值进行判定，如果此函数返回true，接收到的值将在输出流发出，反之，则不会发出。index参数是从开始订阅时，输入流的发出的值的索引，从0开始 |
| thisArg   | any                                    | 可选      | 判定函数运行时的this指向                                                                                                                                       |

返回值

Observable 通过判定函数检测的值组成的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .filter(event => event.target.tagName === 'DIV')
        .subscribe(v => console.log(v));

## first

----
实例方法

发送输入流上的第一个值，或者第一个符合某些条件的值。

* <font face"仿宋">_只发射第一个或第一个符合某些条件的值。_</font>

    ---------a-------b------c---------d-->

                first

    ---------a|

在不传入任何参数时，这个操作符仅发出输入流上的第一个值，然后立即发出结束通知。如果传入一个判定函数，则发出第一个通过判定函数检测的值。它还可以接受一个结果控制函数来转化输出的值，或一个在输入流没有发出符合条件的值情况下使用的默认值。如果没有提供默认值，并且在输入流上也没有找到符合条件的值时，输出流将会抛出错误。

| Name           | Type                                   | Attribute | Description                                                                                                       |
| -------------- | :------------------------------------: | :-------: | ----------------------------------------------------------------------------------------------------------------: |
| predicate      | function(v: T, index: number): boolean | 可选      | 判定函数。用来检测输入流上的值是否符合条件                                                                        |
| resultSelector | function(v: T, index: number): R       | 可选      | 根据输入流上的值生产输出值的函数。这个函数将接收到2个参数：value是输入流输出的数据，index是此数据在输入流上的索引 |
| defaultValue   | R                                      | 可选      | 在输入流上没有找到合法值的情况下使用的默认值                                                                      |

返回值

Observable<T|R> 第一个符合条件的值。

异常

EmptyError 在结束通知发出前如果没有发出过有效值，将会发送一个错误通知给观察者。

示例

    Rx.Observable.fromEvent(document, 'click')
        .first()
        .subscribe(v => console.log(v));
----
    Rx.Observable.fromEvent(document, 'click')
        .first(event => event.target.tagName === 'DIV')
        .subscribe(v => console.log(v));

## ignoreElements

----
实例方法

忽略输入流上所有的正常值，只发出其错误通知和完成通知。

    ------a-----b-----c-----d-------|--->

                ignoreElements

    --------------------------------|--->

返回值

Observable 只能发出错误通知或完成通知的流，具体发出哪一个要取决于输入流。

示例

    const source = Rx.Observable.interval(1000)
        .take(5)
        .map(v => {
            if(v < 4) {
                return v;
            }else {
                throw 'too big';
            }
        });

    source.ignoreElements().subscribe(v => console.log(v), e => console.error(e), () => console.log('complete'));

## audit

----
实例方法

在一段时间内忽略输入流上的值，持续时间由另一条流决定，之后发出输入流上的最近的值。

* <font face="仿宋">_类似于auditTime，只是它的静默时间是由另外一条流决定的。_</font>

    -------a--x-y-------------b---x----------c-x-x-x-----|-->

    -------------|-->

                audit

    -------------y-----------------x----------------x----|-->

这个操作符和throttler类似，只不是它输出的是静默时间过后最近的值，而不是第一个值。audit内部的计时器被禁用后，它会发出输入流上发出的最新的值，在计时器开启时输入流上发出的值都会被忽略，最初的时候计时器是被禁用的。当输入流上的第一个值到达时，传入的函数会被调用，计时器此时开启，此函数会返回一个流，这个流发出值或完成通知后，计时器会停止，然后输入流上发出的最新值将会在输出流上输出。下一个值到达时重复此过程。

参数

| Name             | Type                                      | Attribute | Description                                 |
| ---------------- | :---------------------------------------: | :-------: | ------------------------------------------: |
| durationSelector | function(value: T): SubscribableOrPromise |           | 接受输入流的值作为参数，输出一个流或promise |

返回值

Observable 把输入流的值发射频率限制后的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .audit(event => Rx.Observable.interval(1000))
        .subscribe(v => console.log(v));

## auditTime

----
实例方法

在一段时间内忽略输入流上的值，之后发出输入流上最新的值。

* <font face="仿宋">_当输入流上发出值是，输出流忽略这个值并在接下来的指定时间内忽略输入流的值，过后发出输入流最新的值。_</font>

    --------a--x--y------b---------x----c-x------|-->

                auditTime(500);

    --------------y----------------x-------------|-->

此操作符和 throttleTime 类似，不同的是它在指定的时间段后发出的是输入流上最新的值而不是第一个值。auditTime内部的计时器被禁用后，它会发出输入流上发出的最新的值，在计时器开启时输入流上发出的值都会被忽略，最初的时候计时器是被禁用的。当输入流上的第一个值到达时，计时器此时开启。在指定的时间过后，计时器会停止，然后输入流上发出的最新值将会在输出流上输出。下一个值到达时重复此过程。可以传入一个可选的调度器来管理计时器。

参数

| Name      | Type      | Attribute           | Description                            |
| --------- | :-------: | :-----------------: | -------------------------------------: |
| duration  | number    |                     | 在发出输入流上的最新值时需要等待的时间 |
| scheduler | Scheduler | 可选；默认值：async | 管理计时器，处理发送频率               |

返回值

Observable 把输入流的值发射频率限制后的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .auditTime(1000)
        .subscribe(x => console.log(x));

## last

----
实例方法

输入流上发出输入流上的最后一个值。可以传入一个判定函数，这种情况下，输出流发出的值将会是输入流上满足这个判定函数的最后一个值。

    ----a---------b-------c-------d--|-->

                last

    ---------------------------------d|

参数

| Name      | Type     | Attribute | Description |
| --------- | :------: | :-------: | ----------: |
| predicate | function | 可选      | 判定函数    |

返回值

Observable 发出输入流上符合条件的值的流。

异常

EmptyError 如果在输入流发出结束通知前没有发出过值，或没有发出过符合判定函数条件的值时，向观察者通知一个异常。

示例

    Rx.Observable.of(1,2,3,4)
        .last()
        .subscribe(v => console.log(v));
----
    Rx.Observable.of(1,2,3,4)
        .last(v => v%2 ===1)
        .subscribe(v => console.log(v));

## sample

----
实例方法

当另一个流发出值时输出输入流上最新的值。

* <font face="仿宋">_和sampleTime类似，只不过取样的时机取决于另一条流。_</font>

    -------a------b----c---------------------d--------|-->

    ---------x-----------x-----x----------------x--------|->

                sample

    ---------a-----------c----------------------d--------|->

当通知流上发出值或者完成通知时，此操作符都会查看输入流上的最新值，如果这个值和输出流的前一个值不同，则这个值就会被传递给输出流。在订阅输出流时，通知流同时也会被订阅。

参数

| Name     | Type            | Attribute | Description      |
| -------- | :-------------: | :-------: | ---------------: |
| notifier | Observable<any> |           | 取样时机的通知流 |

返回值

Observable 发出取样结果的流。

示例

    Rx.Observable.interval(1000)
        .sample(Rx.Observable.fromEvent(document, 'click'))
        .subscribe(v => console.log(v));

## sampleTime

----
实例方法

每隔一断时间对输入流的值进行取样。

* <font face="仿宋">_周期性的对输入流进行取样。_</font>

    -------a-----b-c---------d--e------------f-g-d-m--------|-->

                sampleTime(700);

    ----------------c----------------e------------------m--|->

此操作符周期性的查看输入流，并将其最新值作为取样结果传递给输出流，如果自上次取样过后输入流没有发出过数据，则不会传递值。取样行为周期性的发生，并且是在输出流被订阅时就开始。

参数

| Name      | Type      | Attribute           | Description  |
| --------- | :-------: | :-----------------: | -----------: |
| period    | number    |                     | 取样周期     |
| scheduler | Scheduler | 可选；默认值：async | 调度取样时间 |

返回值

Observable 发出取样结果的流。

示例

    Rx.Observable.interval(1000)
        .sampleTime(2000)
        .subscribe(v => console.log(v));

## single

----
实例方法

只输出输入流上唯一一个符合判定条件的值，如果输入流没有符合判定条件的值或者有多个符合条件的值，输出流上将会抛出错误。

    -------a-------b-------c-------|--->

                single

    ----------------X------------------->

参数

| Name      | Type     | Attribute | Description |
| --------- | :------: | :-------: | ----------: |
| predicate | Function |           | 判定函数    |

返回值

Observable 输入流上唯一符合判定条件的值组成的流。

错误

EmptyError 没有符合判定函数的唯一值时发出的错误通知。

示例

    Rx.Observable.interval(1000)
        .take(5)
        .single(v => v === 4)
        .subscribe(v => console.log(v));

## skip

----
实例方法

返回一个跳过指定数量的值的流。

    ---a---b---c---d---e---|->

                skip(3);

    ---------------d---e---|>

参数

| Name  | Type   | Attribute | Description        |
| ----- | :----: | :-------: | -----------------: |
| count | number |           | 需要跳过的值的数量 |

返回值

Observable 跳过了一定数量值的流。

示例

    Observable.from([1,2,3,4,5,6])
        .skip(3)
        .subscribe(v => console.log(v));

## skipLast

----
实例方法

跳过输入流上最后n个值。

    ----1---2---3---4---5---6-----|-->

                skipLast(3);

    ----1---2---3--|->

输出流上累积足够的队列保存最初的n个值，当接收到更多值时，将从队列的前面取值并在结果队列产生，这将导致值的发射被延迟。

参数

| Name  | Type   | Attribute | Description         |
| ----- | :----: | :-------: | ------------------: |
| count | number |           | 需要跳过的最后n个值 |

返回值

Observable 跳过了输入流上最后n个值的流。

错误

ArgumentOutOfRangeError 当i < 0时，抛出此错误。

示例

        Rx.Observable.range(1,5)
            .skipLast(3)
            .subscribe(v => console.log(v));

## skipUntil

----
实例方法

在通知流发出值之前，忽略输入流上产生的值。

    --a-----b-----c----d-----e-----f---g----h---|-->

    ---------------------x---------------|-->

                skipUntil

    ------------------------e------f---g----h---|-->

参数

| Name     | Type       | Attribute | Description                                |
| -------- | :--------: | :-------: | -----------------------------------------: |
| notifier | Observable |           | 发出通知值使输出流开始镜像输入流上发出的值 |

返回值

Observable 忽略了通知流发出通知之前的所有值的流。

示例

    Rx.Observable.interval(1000)
        .skipUntil(Rx.Observable.fromEvent(document, 'click'))
        .subscribe(v => console.log(v));

## skipWhile

----
实例方法

输入流上的值在通过判定函数的结果为true时将会被输出流忽略，一旦判定函数的结果变为false，输出流就开始镜像输入流。

    ---2---3----4-----5----6----1-----|-->

                skipWhile(x => x < 4);

    ------------4-----5----6----1-----|-->

参数

| Name      | Type     | Attribute | Description              |
| --------- | :------: | :-------: | -----------------------: |
| predicate | Function |           | 检测输入流的值的判定函数 |

返回值

Observable 当判定函数的值变为false时开始镜像输入流的流。

示例

    Rx.Observable.interval(1000)
        .skipWhile(x => x < 4)
        .subscribe(v => console.log(v));

## take

----
实例方法

只发出输入流上指定数量的值的流，从第一个值开始。

* <font face="仿宋">_从第一个值开始发出指定数量的值，然后发出结束通知。_</font>

    ---a------b------c------d-----e---|-->

                take(3);

    ---a------b------c|

输出流仅仅发出了输入流上从第一个值开始的n个值。如果输入流上值的个数小于n，那么所有的值都会被发出。值发射完成后，不管输入流有没有发出结束通知，输出流都会立即发出结束通知。

参数

| Name  | Type   | Attribute | Description                |
| ----- | :----: | :-------: | -------------------------: |
| count | number |           | 输出流上允许发出的最大数量 |

返回值

Observable 发出输入流上从第一个值开始的n个值，或者输入流发出值的个数小于n时发出所有的值的流。

异常

ArgumentOutOfRangeError 在给此操作符传入负数时给观察者发了的错误。

示例

    Rx.Observable.interval(1000)
        .take(5)
        .subscribe(v => console.log(v));

## takeLast

----
实例方法

只输出输入流最后发出的n个值。

* <font face="仿宋">_储存输入流上最后发出的n个值，在输入流完成后，输出流是发出这些值。_</font>

    -----a----b---------c-------e----f-----g----h----|--->

                takeLast(2);

    -------------------------------------------------gh|-->

输出流上只输出输入流上最后n个值，如果输入流上的值小于n，那么输入流的值将全部被输出。输出流必须在输入流发出完成通知后才能发出值，因为在此之前无法判断输入流上是否还有值发出，所有的值都以同步的方式输出。

参数

| Name  | Type   | Attribute | Description                |
| ----- | :----: | :-------: | -------------------------: |
| count | number |           | 从输入流上获取值的最大数量 |

返回值

Observable 只输出输入流上最后n个值的流。

错误

ArgumentOutOfRangeError 当n < 0时，此操作符会抛出此错误。

示例

    Rx.Observable.range(1,100)
        .takeLast(3)
        .subscribe(v => console.log(v));

## takeUntil

----
实例方法

在通知流发出通知之前，持续发射输入流上的值。

* <font face="仿宋">_发射输入流上的值，直到通知流上发出一个值，然后发出完成通知。_</font>

在通知流发出值之前，输出流完全就是输入流的镜像。此操作符会一直监视传入的通知流，如果通知流发出了值或结束通知，输出流就会停止发射输入流上的值，并发出完成通知。

| Name     | Type       | Attribute | Description                                        |
| -------- | :--------: | :-------: | -------------------------------------------------: |
| notifier | Observable |           | 通知流。当它发射出一个值以后，输出流将会停止发射值 |

返回值
Observable 持续发出输入流上的值，直到通知流上发出值为止。

示例

    Rx.Observable.interval(1000)
        .takeUntil(Rx.Observable.fromEvent(document, 'click'))
        .subscribe(v => console.log(v));

## takeWhile

----
实例方法

在输入流上的值满足判定条件的情况下持续发射值，一旦某一个值不满足判定条件，立即发出结束通知。

* <font face="仿宋">_在值满足条件时持续发送值，一旦不满足，立即结束。_</font>

    ---2---3----4---5---7---|-->

                takeWhile(x => x < 5);

    ---2---3----4---|

这个操作会订阅输入流，在满足判定条件时发出的完全是输入流的镜像。判定函数返回的布尔值代表条件是否满足，一旦此函数返回false，输入流将立即停止镜像并发出完成通知。

参数

| Name      | Type                                       | Attribute | Description                                                                |
| --------- | :----------------------------------------: | :-------: | -------------------------------------------------------------------------: |
| predicate | function(value: T, index: number): boolean |           | 判定函数。判定输入流上的值是否满足给定的条件，第二个参数是输入流上值的索引 |

返回值

Observable 满足判定条件时的输入流镜像，不满足时立即结束。

示例

    Rx.Observable.fromEvent(document, 'click')
        .takeWhile(event => event.clientX > 200)
        .subscribe(v => console.log(v));

## throttle

----
实例方法

发出输入流上的某一个值，然后在指定时间内忽略输入流上发出的值，这个指定时间由另外一流决定。

* <font face="仿宋">_和throttleTime类似，只不过静默时间是由另一条流决定的。_</font>

    --a--x-y---------b---x-----------c--x-x-x-x--------|-->

    ------|->

                throttle

    --a--------------b---------------c-----------------|-->

当计时器被禁用后在输出流上发出输入流的值，计时器开启时输入上的值都会被忽略。计时器在最初是被禁用的，当输入流的第一个值到达时，它会直接被输出流输出，然后传入的函数被调用，计时器开启，这个函数会返回一个流来决定静默时间。当返回的流产生了值或者发出完成通知时，计时器被禁用。在下一值到达时重复以上过程。

参数

| Name             | Type                                      | Attribute | Description                                                                             |
| ---------------- | :---------------------------------------: | :-------: | --------------------------------------------------------------------------------------: |
| durationSelector | function(value: T): SubscribableOrPromise |           | 接受输入流的值作为参数，输出一个流或promise                                             |
| config           | Object                                    |           | 用来定义 leading 和 trailing 行为的配置对象。 默认为 { leading: true, trailing: false } |

返回值

Observable 对输入流进行了节流包装及频率限制的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .throttle(event => Rx.Obsevable.interval(1000))
        .subscribe(v => console.log(v));

## throttleTime

----
实例方法

发出输入流上的某一个值，然后在指定时间内忽略输入流上发出的值。

* <font face="仿宋">_让一个值通过，然后在接下来的指定时间内忽略输入流上的值。_</font>

    --a--x-y---------b---x-----------c--x-x-x-x--------|-->

                throttleTime(500)

    --a--------------b---------------c-----------------|-->

当计时器被禁用后在输出流上发出输入流的值，计时器开启时输入上的值都会被忽略。计时器在最初是被禁用的，当输入流的第一个值到达时，它会直接被输出流输出，然后计时器开启，在经过指定的静默时间后，计时器被禁用。在下一值到达时重复以上过程。可以传入一个可选的Scheduler管理计时器。

参数

| Name      | Type      | Attribute           | Description                            |
| --------- | :-------: | :-----------------: | -------------------------------------: |
| duration  | number    |                     | 在发出输入流上的最新值时需要等待的时间 |
| scheduler | Scheduler | 可选；默认值：async | 管理计时器，处理发送频率               |

返回值

Observable 对输入流进行了节流包装及频率限制的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .throttleTime(500)
        .subscribe(v => console.log(v));
