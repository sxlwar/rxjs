# 过滤类型操作符

## debounce

## debounceTime

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

## distinctKey

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

## audit

## auditTime

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

## sampleTime

## single

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

## skipUntil

## skipWhile

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

## throttleTime
