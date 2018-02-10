# 转换类型操作符

## buffer

## bufferCount

----
实例方法

缓冲输入流中的值，直到输入流中的值到达指定数量时，才在输出流中发出。

* <font face="仿宋">_将输入流中的值收集到数组中，当数组的长度到达指定的长度时将其发出。_</font>

    --1--2--3--4--5--6--7--8--9--10-|->

                bufferCount(3,2);

    --------[1,2,3]--------[4,5,6]--------[7,8,9]---[10]|

缓冲输入流上的值，当值的数量达到指定的数量时，发出该缓存值，并清理缓冲区。第二个参数决定缓冲区开启的频次，如果不提供第二个参数或者其值为null时，每次开始缓冲时都会开启新的缓冲区，在值发射过后清空。

| Name             | Type   | Attribute | Description                                                                                           |
| ---------------- | :----: | :-------: | ----------------------------------------------------------------------------------------------------: |
| bufferSize       | number |           | 缓冲区储存值的最大数量                                                                                |
| startBufferEvery | number | 可选      | 开启新缓冲区的频次，例如当这个值为2时，隔一个数据会开启一次缓冲区。默认情况下，开始发射值时开启缓冲区 |

返回值

Observable 以数组形式来缓存值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .debounceCount(2,2)
        .subscribe(v => console.log(v));

## bufferTime

## bufferToggle

## bufferWhen

## concatMap

## concatMapTo

## exhaustMap

## expand

## groupBy

----
实例方法

将输入流上的值按一定的规则分组成不同的流，然后把这些流发送给输出流，每一条流上都是一组符合相同条件的值。

    ----1----2----3----4----5----|->

                groupBy(v => v%2);

    -----------------------------|->
        \   \
         \   2---------4---------|
          1------3----------5----|

参数

| Name             | Type                                                     | Attribute | Description                                |
| ---------------- | :------------------------------------------------------: | :-------: | -----------------------------------------: |
| keySelector      | function(value:T):K                                      |           | 分组函数，用于检查输入流上的每一个值       |
| elementSelector  | function(value: T):R                                     | 可选      | 用来检查每一个返回值的函数                 |
| durationSelector | function(grouped:GroupedObservable<K,R>):Observable<any> | 可选      | 返回一个流，用来决定每个组应该存在多长时间 |

返回值

Observable<GroupedObservable<K,R>> 发出分组后的流的高阶流，分组的流都有一个唯一的key，并且该流中的值都是输入流上符合某一条件的值。

示例

    Rx.Observable.of(
        {id: 1, name: 'aze1'},
        {id: 2, name: 'sf2'},
        {id: 2, name: 'dg2'},
        {id: 1, name: 'erg1'},
        {id: 1, name: 'df1'},
        {id: 2, name: 'sf2'},
        {id: 3, name: 'qfs3'},
        {id: 2, name: 'qsg'}
    )
    .groupBy(v => v.id)
    .mergeMap(group => group.reduce((acc,cur) => [...acc, cur],[]))
    .subscribe(v => console.log(v));

## map

----
实例方法

输出流上的值是使用一个映射函数将输入流上的值映射后得到新的值。

* <font face="仿宋">_和数组的map方法行为一样，将输入的值根据规则进行转换后在输入流上发出。_</font>

    --------1-----------2----------3----------|--->

                map(v => v * 10);

    --------10----------20---------30---------|---->

参数

| Name    | Type                                 | Attribute | Description                                                                                       |
| ------- | :----------------------------------: | :-------: | ------------------------------------------------------------------------------------------------: |
| project | function(value: T, index: number): R |           | 映射函数，第一个参数是从输入流上得到的值，第二个参数是这个值在开始订阅后在输入流上的索引，从0开始 |
| thisArg | any                                  | optional  | 映射函数运行时的this指向                                                                          |

返回值

Observable 发出映射后的值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .map(event => event.clientX)
        .subscribe(v => console.log(v));

## mapTo

----
实例方法

当输入源上发出值时，把它映射成一个一常数在输出流上输出。

* <font face="仿宋">_行为与map一样，只不过每次映射的结果都一样_</font>

    -----1--------2---------3-----|-->

                mapTo('a');

    -----a--------a---------a-----|-->

接受一个常量作为参数，每当输入流上有值发出时，输出流都发出这个常量，换句话讲就是它只关心输入流什么时候发出值，而不关心输入流发出的值本身。

参数

| Name  | Type  | Attribute | Description |
| ----- | :---: | :-------: | ----------: |
| value | any   |           | 输出的值    |

返回值

Observable 每次在输入流发出值时都发出同样值的流。

示例

    Rx.Observable.fromEvent(document,'click')
        .mapTo('Hello')
        .subscribe(v => console.log(v));

## mergeMap

----
实例方法

将所有输入流的值都合并到一条流上。

* <font face="仿宋">_映射每一条输入流，并将它们打平到一条流上。_</font>

    -1-----2-----3---------|-->

    ---2----2----2--|-----> //第一次时的内部流，第2，3次的一样，这个流是直正的输入流

                mergeMap(v => Observable.of(v + 1).repeat(3));

    -2--2--2--3--3--3--4--4--4----|-->

输出流会把所有从映射函数中返回的内部流打平到一条流上。映射函数可以使用输入流的值来生成内部流。

参数

| Name           | Type                                                                                | Attribute                              | Description                                                                                                                                                    |
| -------------- | :---------------------------------------------------------------------------------: | :------------------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| project        | function(value: T, index: number): ObservableInput                                  |                                        | 接受输入流参数，返回内部的流                                                                                                                                   |
| resultSelector | function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): nay | 可选                                   | 基于输入流和内部流的值和逻辑来生成值的函数。 outerValue: 输入流的上的值 ;innerValue: 内部流上的值; outerIndex: 输入流的值的索引; innerIndex: 内部流的值的索引; |
| concurrent     | number                                                                              | 可选。默认值：Number.POSITIVE_INFINITY | 同时可以订阅的内部流的最大数量                                                                                                                                 |

返回值

Observable 一条合并了所有映射函数生成的内部流的流，内部流上的值都会在这条流上发出。

示例

    Rx.Observable.of('a','b','c')
        .mergeMap(v => Rx.Observable.interval(1000).map(x => x + v ))
        .subscribe(v => console.log(v));

----
    const source = Rx.Observable.of('Hello');

    const createPromise = v => new Promise(resolve => resolve(`I got ${v} from promise`));

    source.mergeMap(
        v => createPromise(v),
        (outValue, innerValue) => `Source: ${outValue},${innerValue}`
    )
    .subscribe(v => console.log(v));

## mergeMapTo

----
实例方法

把输入流的值都映射成同样的流，然后把这些映射后的流进行合并。

* <font face="仿宋">_和mergeMap的行为类似，只不过得到的内部流都是相同的。_</font>

    ------1----2------3------|-->

    --10--10--10--|--> //内部流

                mergeMapTo

    ------10--10--10----10--10--10-----10--10--10--|-->

参数

| Name           | Type                                                                                | Attribute                              | Description                                                                                                                                                    |
| -------------- | :---------------------------------------------------------------------------------: | :------------------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| project        | function(value: T, index: number): ObservableInput                                  |                                        | 接受输入流参数，返回内部的流                                                                                                                                   |
| resultSelector | function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): nay | 可选                                   | 基于输入流和内部流的值和逻辑来生成值的函数。 outerValue: 输入流的上的值 ;innerValue: 内部流上的值; outerIndex: 输入流的值的索引; innerIndex: 内部流的值的索引; |
| concurrent     | number                                                                              | 可选。默认值：Number.POSITIVE_INFINITY | 同时可以订阅的内部流的最大数量                                                                                                                                 |

返回值

Observable 一条合并了所有映射函数生成的内部流的流，内部流上的值都会在这条流上发出。

示例

    Rx.Observable.fromEvent(document, 'click')
        .mergeMapTo(Rx.Observable.interval(1000).take(5))
        .subscribe(v => console.log(v));

## mergeScan

## pairwise

## partition

## pluck

----
实例方法

指定一个目标属性，抽取输入流发出的值的此属性。

* <font face="仿宋">_和map方法类似，但只能用来提取输入流出值的一个内部属性。_</font>

    -----{a:1}-----{a:2}-----{a:3}----|-->

                pluck('a');

    ------1---------2----------3------|>

获取输入流发出的值的属性值，接收的参数作为属性查找的路径，然后使用这个路径在每一个值上查找其属性值，如果没有找到，则会发出undefined。

参数

| Name     | Type      | Attribute | Description          |
| -------- | :-------: | :-------: | -------------------: |
| property | ...string |           | 需要查找的目标属性值 |

返回值

Observable 发出提取的值组成的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .pluck('target', 'tagName')
        .subscribe(v => console.log(v));

## scan

----
实例方法

在输入流上应用一个累积函数，每一次累积的结果都发送给输出流，累积时可以接受一个可选的初始值。

* <font face="仿宋">_和reduce操作符类似，不同的是会把每一次累积的结果都发送出去。_</font>

将输入流发出的值能过累积函数进行累积，累积函数知道如何把新接收到的值加入已有的累积结果中。如果提供了初始值，那么它将作为整个累积过程的初始值，如果没有提供，输入流上的第一个值将作为整个累积过程的初始值。

参数

| Name        | Type                         | Attribute | Description                  |
| ----------- | :--------------------------: | :-------: | ---------------------------: |
| accumulator | function(acc: R, value: T):R |           | 累积函数，用于累积每个输入值 |
| value       | T                            | R         | 可选的初始值                 |

返回值

Observable<R> 输出累积值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .mapTo(1)
        .scan((acc,cur) => acc + cur, 0)
        .subscribe(v => console.log(v));

## switchMap

----
实例方法

在输入流发出值时，把它映射成一条内部流，然后把这条内部流打平成到输出流上，输出流上只会发出最近的内部流上的值。

* <font face="仿宋">_把输入流上的值映射成流，然后使用switch操作符把内部流打平到输出流上。_</font>

    -1---------3-----5----|->

    -10---10---10-| // 内部流

                switchMap(v => Observable.from([10,10,10]).map(x => x * v))

    -10---10---10-30---30-50---50---50-|

输出流上的值是由映射函数在输入流的值的基本上生成的内部流所发出的，输出流只允许观察一条内部流，当有新的内部流到达时，输出流将会取消对之前的内部流的订阅，转而订阅这个最新的内部流，并发出它上面的值。

参数

| Name           | Type                                                                                | Attribute                              | Description                                                                                                                                                    |
| -------------- | :---------------------------------------------------------------------------------: | :------------------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| project        | function(value: T, index: number): ObservableInput                                  |                                        | 接受输入流参数，返回内部的流                                                                                                                                   |
| resultSelector | function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): nay | 可选                                   | 基于输入流和内部流的值和逻辑来生成值的函数。 outerValue: 输入流的上的值 ;innerValue: 内部流上的值; outerIndex: 输入流的值的索引; innerIndex: 内部流的值的索引; |
| concurrent     | number                                                                              | 可选。默认值：Number.POSITIVE_INFINITY | 同时可以订阅的内部流的最大数量                                                                                                                                 |

返回值

Observable 仅从最新的内部流上取值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .switchMap(event => Rx.Observable.interval(1000))
        .subscribe(v => console.log(v));

## switchMapTo

----
实例方法

将输入流上的值都转换成一个内部流，然后将它打平到输出流上。

* <font face="仿宋">_它的行为类似于switchMap，不同的时它映射出的内部流都是相同的。_</font>

    -1---------3-----5----|->

    -6---6---6-| // 内部流

                switchMapTo

    -6---6---6-6---6-6---6---6-|

输出流上的值是由映射函数生成的内部流所发出的，输出流只允许观察一条内部流，当有新的内部流到达时，输出流将会取消对之前的内部流的订阅，转而订阅这个最新的内部流，并发出它上面的值。

| Name            | Type                                                                                | Attribute | Description                                                                                                                                                    |
| --------------- | :---------------------------------------------------------------------------------: | :-------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| innerObservable | ObservableInput                                                                     |           | 内部流                                                                                                                                                         |
| resultSelector  | function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): nay | 可选      | 基于输入流和内部流的值和逻辑来生成值的函数。 outerValue: 输入流的上的值 ;innerValue: 内部流上的值; outerIndex: 输入流的值的索引; innerIndex: 内部流的值的索引; |

返回值

Observable 只能发出最新内部流的值，或者经过resultSelector处理过的值，新的内部流到达时会取消旧的订阅。

示例

    Rx.Observable.fromEvent(document,'click')
        .switchMapTo(Rx.Observable.interval(1000))
        .subscribe(x => console.log(v));

## window

## windowCount

## windowTime

## windowToggle

## windowWhen