# 转换类型操作符

## buffer

----
实例方法

缓冲输入流上发出的值，直到收到缓冲结束通知时再把缓冲值一次性发出。

* <font face="仿宋">_在输入流上收集值，当通知流上发出通知时发出收集到的这些值。_</font>

    --a--b--c--d--e--f--g--h--i--j--k--l--m--n--o--p--q--|-->

    -----------------n-----------------n--------------n--|-->

                buffer

    ----------------[a,b,c,d,e,f]--------------------[g,h,i,j,k,l]-----------------[m,n,o,p,q]--|-->

缓冲输入流上的值，等到通知流发出结束缓冲的通知时，发出这些值，然后再开始下一次的缓冲。

参数

| Name         | Type            | Attribute | Description |
| ------------ | :-------------: | :-------: | ----------: |
| closeNotifer | Observable<any> |           | 通知流      |

返回值

Observable 发出缓冲值的流，缓冲值以数组的形式保存。

示例

    Rx.Observable.interval(1000)
        .buffer(Rx.Observable.fromEvent(document, 'click'))
        .subscribe(v => console.log(v));

## bufferCount

----
实例方法

缓冲输入流中的值，直到输入流中的值到达指定数量时，才在输出流中发出。

* <font face="仿宋">_将输入流中的值收集到数组中，当数组的长度到达指定的长度时将其发出。_</font>

    --1--2--3--4--5--6--7--8--9--10-|->

                bufferCount(3,2);

    --------[1,2,3]--------[4,5,6]--------[7,8,9]---[10]|

缓冲输入流上的值，当值的数量达到指定的数量时，发出该缓存值，并清理缓冲区。第二个参数决定缓冲区开启的频次，如果不提供第二个参数或者其值为null时，每次开始缓冲时都会开启新的缓冲区，在值发射过后清空。

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

----
实例方法

缓冲指定时间周期内输入流上发出的值。

* <font face="仿宋">_以数组的形式收集输入流上的值，每隔一定的周期发出这些值。_</font>

    ---a----b----c----d----e----f----|-->

                bufferTime(200);

    --------[a,b]--------[c,d]---------[e,f]----|-->

如果不提供可选参数bufferCreationInterval，每个时间周期过后都会重置缓冲区，如果提供了这个参数，这个操作符会以bufferCreationInterval指定的时间来开启新的缓冲区，以bufferTimeSpan指定的时间结束缓冲。当指定maxBufferSize参数时，缓冲区在时间到达或缓冲数量达到最大值时都会结束。

参数

| Name                   | Type      | Attribute         | Description            |
| ---------------------- | :-------: | :---------------: | ---------------------: |
| bufferTimeSpan         | number    |                   | 发出缓冲值的周期       |
| bufferCreationInterval | number    | 可选              | 开启新的缓冲的时间间隔 |
| maxBufferSize          | number    | 可选              | 缓冲的最大数量         |
| scheduler              | Scheduler | 可选；默认：async | 调节缓冲区             |

返回值

Observable 输入缓冲值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .map(event => event.clientX)
        .bufferTime(1000)
        .subscribe(v => console.log(v));

----

    Rx.Observable.fromEvent(document, 'click')
        .map(event => event.clientX)
        .bufferTime(2000, 5000)
        .subscribe(v => console.log(v));

## bufferToggle

----
实例方法

缓冲输入流的值，openings 发送的时候开始缓冲，closingSelector 发送的时候结束缓冲。

* <font face="仿宋">_在opening发射值的时候开始缓冲数据，然后调用 closingSelector 函数获取通知关系缓冲的流。_</font>

    ---a---b---c---d---e---f---g---h---i---|-->

    -o---------------------------o---------|-->

    -------------c-------------------c-----|-->

                bufferToggle

    -------------[a,b,c]-------------[h]---|-->

在 opening 发出值时打开缓冲区，开始缓冲输入流的数据，在 selectorFunction 返回的流或者 promise 发出值时再结束缓冲，然后将缓冲得到的值传递给输出流。

参数

| Name            | Type                    | Attribute | Description                                                     |
| --------------- | :---------------------: | :-------: | --------------------------------------------------------------: |
| opeings         | Subscribalbe 或 Promise |           | 通知何时开始缓冲                                                |
| closingSelector | Subscribalbe 或 Promise | 可选      | 接受openings发出的值作为参数，返回流或Promise，通知何时关闭缓冲 |

返回值

Observable 发出缓冲值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .map(event => event.clientY)
        .bufferToggle(
            Rx.Observable.interval(5000),
            i => i % 2 ? Rx.Observable.interval(2000) : Rx.Observable.empty()
        )
        .subscribe(v => console.log(v));

## bufferWhen

----
实例方法

缓冲输入流上的值，使用一个工厂函数来获取结束缓冲的通知流。

* <fonct face="仿宋">_缓冲输入流上的数据，当收集开始时调用传入的函数获取结束缓冲的通知流。_</font>

    ---a---b---c---d---e---f---g--|->

    -------------s|->

                bufferWhen

    -------------[a,b,c]--------[d,e,f,g]--[]|->

立即开启缓冲，在输入函数返回的流发出值是结束缓冲并在输出流上发出缓冲值，紧接着开始下一次缓冲过程。

参数

| Name            | Type                  | Attribute                            | Description |
| --------------- | :-------------------: | :----------------------------------: | ----------: |
| closingSelector | function():Observable | 返回缓冲结束的通知流，不接受任何参数 |

返回值

Observable 发出缓冲值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .map(event => event.clientX)
        .bufferWhen(() => Rx.Observable.interval(1000 + Math.random() * 4000))
        .subscribe(x => console.log(x));

## concatMap

----
实例方法

将多个流合并到一条流上，需要等待当前的流完成后才能开始合并下一个，合并过程按传入的顺序进行。

* <font face="仿宋">_将每一个值映射成Observable，然后使用concatlAll将所有的内部流打平。_</font>

    --1------------3---------5-----------|-->

    --10---10---10--|-->

                concatMap(i => 10*i----10*i---10*i)

    --10---10---10-30---30---30-50---50---50--|->

把输入流的值通过传入的函数映射成流，此函数返回的流将按次序进行连接。

注意：如果输入流的值不断的发送出来，并且速度快于内部流完成的速度，将会导致内部问题，因为内部流无限的在缓冲区聚集等待轮流订阅。

concatMap 等价于给并发数量为1的 mergeMap。

参数

| Name           | Type                                                                              | Attribute | Description                                                                                           |
| -------------- | :-------------------------------------------------------------------------------: | :-------: | ----------------------------------------------------------------------------------------------------: |
| project        | function(value: T, index?:number):ObservableInput                                 |           | 接收输入流的值，返回内部流的函数                                                                      |
| resultSelector | function(outerValue: T, innerValue: I, outerIndex:number, innerIndex: number):any | 可选      | 它用于产生基于值的输出 Observable 和源(外部)发送和内部 Observable 发送的索引。 传递给这个函数参数有： |
outerValue: 来自源的值
innerValue: 来自投射的 Observable 的值
outerIndex: 来自源的值的 "index"
innerIndex: 来自投射的 Observable 的值的 "index" |

返回值

Observable 把输入的值经过映射成流后再连接起来的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .concatMap(event => Rx.Observable.interval(1000).take(3))
        .subscribe(v => console.log(v));

## concatMapTo

----
实例方法

将输入源流上的值映射成多个相同的流并将它们按先后顺序合并到一条流上。

* <font face="仿宋">_类似于concatMap，但是每次映射成的内部流都是相同的。_</font>

    --1------------3---------5-----------|-->

    --10---10---10--|-->

                concatMap(10----10---10-|)

    --10---10---10-10---10---10-10---10---10--|->

把输入流的值通过传入的函数映射成相同的内部流，此函数返回的流将按次序进行连接。

注意：如果输入流的值不断的发送出来，并且速度快于内部流完成的速度，将会导致内部问题，因为内部流无限的在缓冲区聚集等待轮流订阅。

concatMapTo 等价于给并发数量为1的 mergeMapTo。

参数

| Name            | Type                                                                              | Attribute | Description                                                                                           |
| --------------- | :-------------------------------------------------------------------------------: | :-------: | ----------------------------------------------------------------------------------------------------: |
| innerObservable | ObservableInput                                                                   |           | 替换输入流上的值的内部流                                                                              |
| resultSelector  | function(outerValue: T, innerValue: I, outerIndex:number, innerIndex: number):any | 可选      | 它用于产生基于值的输出 Observable 和源(外部)发送和内部 Observable 发送的索引。 传递给这个函数参数有： |
outerValue: 来自源的值
innerValue: 来自投射的 Observable 的值
outerIndex: 来自源的值的 "index"
innerIndex: 来自投射的 Observable 的值的 "index" |

返回值

Observable 把输入的值经过映射成流后再连接起来的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .concatMapTo(Rx.Observable.interval(1000).take(3))
        .subscribe(v => console.log(v));

## exhaustMap

----
实例方法

将输入流上的值输入成流，然后将这个条流上的值合并到输出流上，只有当前一个流完成时，新的内部流的值才会开始合并。

* <font face="仿宋">_将输入流的值映射成流，然后能过exhaust将其打平。_</font>

    ---1--------5--------3--------5-----------------|-->

    ---10-----10-----10-|->

                exhaustMap(i => 10*i----10*i----10*i);

    ---10-----10-----10--10-----10-----10---|-->

将输入流的值通过传入的函数转化后得到一条内部流，然后把内部流打平到输出流上。当新的内部流到达而之前的内部流尚未发出完成通知时，新的内部流将会被忽略，也就是说只有当前的内部流发出完成通知后接收到的新的内部流才会继续合并。

参数

| Name    | Type                                             | Attribute | Description                |
| ------- | :----------------------------------------------: | :-------: | -------------------------: |
| project | function(value:T, index?:number):ObservableInput |           | 接收输入流的值，返回内部流 |

| resultSelector  | function(outerValue: T, innerValue: I, outerIndex:number, innerIndex: number):any | 可选      | 它用于产生基于值的输出 Observable 和源(外部)发送和内部 Observable 发送的索引。 传递给这个函数参数有： |
outerValue: 来自源的值
innerValue: 来自投射的 Observable 的值
outerIndex: 来自源的值的 "index"
innerIndex: 来自投射的 Observable 的值的 "index" |\

返回值

Observable 由内部流根据上述规则组成的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .exhaustMap(event => Rx.Observable.interval(1000).take(3))
        .subscribe(v => console.log(v));

## expand

----
实例方法

将输入流上的值递归转化成流再合并到输出流上。

* <font face="仿宋">_类似于mergeMap，但是映射函数会被用于每个输入值及每一个输出值。_</font>

    ------1------------------|-->

        -------2---------|----->

                expand(x => x === 8 ? empty: ----2*x---|);

    -----1----2-----4-----8----|-->

输入的函数接收输入流的值作为参数，返回内部流，内部流被合并到输出流上并发出值。当有新的内部流到达时，此操作符会重新发出输出流上的值并对它们应用输入的函数。

参数

| Name       | Type                              | Attribute                                  | Description                                        |
| ---------- | :-------------------------------: | :----------------------------------------: | -------------------------------------------------: |
| project    | function(value: T, index: number) |                                            | 映射函数，接受输入流或输出流的值，将其转化成一条流 |
| concurrent | number                            | optional; 默认值：Number.POSITIVE_INFINITE | 同时订阅的输入流的最大数量                         |
| scheduler  | Shceduler                         | 可选；默认值： null                        | 调度内部流的订阅                                   |

返回值

Observable 对输入流和输出流的值都应用映射函数得到的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .mapTo(1)
        .expand(x => Rx.Observable.of(2*x).delay(1000))
        .take(10)
        .subscribe(x => console.log(x));

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

Observable 一条合并了所有映射函数生成的内部流的流，内部流上的值都会在这条流上发出。

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

把输入流的值都映射成同样的流，然后把这些映射后的流进行合并。

* <font face="仿宋">_和mergeMap的行为类似，只不过得到的内部流都是相同的。_</font>

    ------1----2------3------|-->

    --10--10--10--|--> //内部流

                mergeMapTo

    ------10--10--10----10--10--10-----10--10--10--|-->

参数

| Name           | Type                                                                                | Attribute                              | Description                                                                                                                                                    |
| -------------- | :---------------------------------------------------------------------------------: | :------------------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| project        | function(value: T, index: number): ObservableInput                                  |                                        | 接受输入流参数，返回内部的流                                                                                                                                   |
| resultSelector | function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): nay | 可选                                   | 基于输入流和内部流的值和逻辑来生成值的函数。 outerValue: 输入流的上的值 ;innerValue: 内部流上的值; outerIndex: 输入流的值的索引; innerIndex: 内部流的值的索引; |
| concurrent     | number                                                                              | 可选。默认值：Number.POSITIVE_INFINITY | 同时可以订阅的内部流的最大数量                                                                                                                                 |

返回值

Observable 一条合并了所有映射函数生成的内部流的流，内部流上的值都会在这条流上发出。

示例

    Rx.Observable.fromEvent(document, 'click')
        .mergeMapTo(Rx.Observable.interval(1000).take(5))
        .subscribe(v => console.log(v));

## mergeScan

----
实例方法

在输入流上应用累积函数，累积函数返回内部流 ，然后每个返回的内部流被合并到输出输出流中。

* <font face="仿宋">_类似于scan，但由累积函数返回的内部流会被合并到输出流中。_</font>

参数

| Name        | Type                                  | Attribute                               | Description                  |
| ----------- | :-----------------------------------: | :-------------------------------------: | ---------------------------: |
| accumulator | function(acc:R,value:T):Observable<R> |                                         | 累积函数                     |
| seed        | any                                   |                                         | 累积的初始值                 |
| concurrent  | number                                | 可选的 默认值：Number.POSITIVE_INFINITY | 可周时订阅的输入流的最大数量 |

返回值

Observable 累积后的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .mapTo(1)
        .mergeScan((acc, cur) => Rx.Observable.fo(acc + cur), 0)
        .subscribe(v => console.log(v));

## pairwise

----
实例方法

将一系列连续发射的值成对的组合在一起，发出这些组合后的值。

* <font face="仿宋">_将当前值和前一个值组合后发出。_</font>

    ---a----b-----c-----d-------|-->

                pairwise

    -------[a,b]--[b,c]-[c,d]--|-->

一条发出n个值的流将被转化成一条发出n-1组值的流，每组值都是包含2个值的数组。因此只有当输入流开始发出第二个值时，输出流上才开始发出值。

返回值

Observable 发出成对值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .pairwise()
        .map(pair => {
            const x0 = pair[0].clientX;
            const y0 = pair[0].clientX;
            const x1 = pair[1].clientY;
            const y1 = pair[1].clientY;

            return Math.sqrt(Map.pow(x0 - x1,2) + Math.pow(y0 - y1,2))
        })
        .subscribe(x => console.log(x));

## partition

----
实例方法

使用判定函数将输入流分成2条不同的流，一条发出满足判定条件的值，一条发出不满足判定条件的值。

* <font face="仿宋">_类似于filter，但它会返回2条不同的流。_</font>

    ---1--2--3--4-----5------6----7--8---|-->

                partition(x => x%2 ===1 );

    ---1-----3--------5-----------7------|-->

    ------2-----4------------6-------8---|-->

此操作符返回一个数组，数组的第一个值是发出满足判定函数的值组成的流，第二个值是不满足判定函数的值组成的流。

参数

| Name      | Type                                       | Attribute | Description                                                                                                                                 |
| --------- | :----------------------------------------: | :-------: | ------------------------------------------------------------------------------------------------------------------------------------------: |
| predicate | function(value: T, index: number): boolean |           | 判定函数，输入流的值通过判定函数检测为true时分配到第一条流上，否则分配在第二条流上。index 参数是自订阅开始后发送序列的索引，是从 0 开始的。 |
| thisArg   | any                                        | 可选的    | 决定判定函数运行时的this指向                                                                                                                |

返回值

[Observable, Observable] 第一个是通过检测的流，第二个未通过检测的流。

示例

    const parts = Rx.Observable.fromEvent(document, 'click')
        .partition(event => event.target.tagName === 'DIV')

    const clickOnDivs = parts[0];

    const clickOnOthers = parts[1];

    clickOnDivs.subscribe(v => console.log(v));

    clickOnOthers.subscribe(v => console.log(v));

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

在输入流发出值时，把它映射成一条内部流，然后把这条内部流打平成到输出流上，输出流上只会发出最近的内部流上的值。

* <font face="仿宋">_把输入流上的值映射成流，然后使用switch操作符把内部流打平到输出流上。_</font>

    -1---------3-----5----|->

    -10---10---10-| // 内部流

                switchMap(v => Observable.from([10,10,10]).map(x => x * v))

    -10---10---10-30---30-50---50---50-|

输出流上的值是由映射函数在输入流的值的基本上生成的内部流所发出的，输出流只允许观察一条内部流，当有新的内部流到达时，输出流将会取消对之前的内部流的订阅，转而订阅这个最新的内部流，并发出它上面的值。

参数

| Name           | Type                                                                                | Attribute                              | Description                                                                                                                                                    |
| -------------- | :---------------------------------------------------------------------------------: | :------------------------------------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| project        | function(value: T, index: number): ObservableInput                                  |                                        | 接受输入流参数，返回内部的流                                                                                                                                   |
| resultSelector | function(outerValue: T, innerValue: I, outerIndex: number, innerIndex: number): nay | 可选                                   | 基于输入流和内部流的值和逻辑来生成值的函数。 outerValue: 输入流的上的值 ;innerValue: 内部流上的值; outerIndex: 输入流的值的索引; innerIndex: 内部流的值的索引; |
| concurrent     | number                                                                              | 可选。默认值：Number.POSITIVE_INFINITY | 同时可以订阅的内部流的最大数量                                                                                                                                 |

返回值

Observable 仅从最新的内部流上取值的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .switchMap(event => Rx.Observable.interval(1000))
        .subscribe(v => console.log(v));

## switchMapTo

----
实例方法

将输入流上的值都转换成一个内部流，然后将它打平到输出流上。

* <font face="仿宋">_它的行为类似于switchMap，不同的时它映射出的内部流都是相同的。_</font>

    -1---------3-----5----|->

    -6---6---6-| // 内部流

                switchMapTo

    -6---6---6-6---6-6---6---6-|

输出流上的值是由映射函数生成的内部流所发出的，输出流只允许观察一条内部流，当有新的内部流到达时，输出流将会取消对之前的内部流的订阅，转而订阅这个最新的内部流，并发出它上面的值。

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

----
实例方法

当分割流发出值后把输入流进行分割，从而把输入流转化成多条内部流。

* <font face="仿宋">_类似于buffer，但发出的是分割后的流，而不是数组。_</font>

    --a--b--c--d--e--f--g--h--i--j--k--l--m--n--|->

    ---------w------------------w---------------|->

                window

    --------------------------------------------|->
    \       \                   \j--k--l--m--n|
     \       \d--e--f--g--h--i|
      \a--b--c|

此操作符返回一个将输入流切分后的流组成的流。每一条切分后的流都发出各自的值，这些值在各个流中不会发生重叠。输出流是一个高阶流。

参数

| Name             | Type            | Attribute | Description      |
| ---------------- | :-------------: | :-------: | ---------------: |
| windowBoundaries | Observable<any> |           | 发出分割通知的流 |

返回值

Observable<Observable<T>> 发出分割后的流的流。

示例

    Rx.Observble.fromEvent(docuement, 'click')
        .window(Rx.Observable.interval(1000))
        .map(win => win.take(2))
        .mergeAll()
        .subscribe(x => console.log(x));

----

    const source = Rx.Observable.timer(0,1000);

    const example = source.windwo(Rx.Observable.interval(3000))

    const count = example.scan((acc,cur) => acc + 1, 0);

    const subscription1 = count.subscribe(v => console.log(`window: ${v}`));

    const subscription2 = example.mergeAll().subscribe(v => console.log(v));

## windowCount

----
实例方法

把输入流进行切分，切分后的流最多可以发出指定数量的值。

* <font face="仿宋">_类似于bufferCount， 但发出的是流，而不是数组。_</font>

    -----a----b-----c-----d----e----f---g---h---|-->

                windowCount(3)

    --------------------------------------------|-->
        \a----b----c|     \d---e---f|   \g---h|

输入流发送由输入流分割后的子流形成的流，每个子流最多可以发出指定数量的值。当输入流上发出完成通知或者错误通知时，输出流把这个通知传递给相应的子流。另外，可以指定开启分割的间隔给此操作符，此时会按指定的间隔来开启新的流，如果没有指定的话，输入流的值到达时会立即开启子流，在子流完成后立即开启下一个子流。

参数

| Name             | Type   | Attribute | Description                    |
| ---------------- | :----: | :-------: | -----------------------------: |
| windowSize       | number |           | 每一个子流可以发出值的最大数量 |
| startWindwoEvery | number | 可选      | 开启子流的间隔                 |

返回值

Observable<Observable<T>> 发出分割后的流的流。

示例

    Rx.Observable.fromEvetn(document, 'click')
        .windwoCount(3)
        .map(win => win.skip(1))
        .mergeAll())
        .subscribe(v => console.log(v));

----

    Rx.Observable.fromEvent(document, 'click')
        .windowCount(2, 3)
        .mergeAll()
        .subscribe(x => console.log(x));

----

    Rx.Observable.interval(1000)
        .windwoCount(4)
        .do(() => console.log('NEW WINDWO'))
        .mergeAll()
        .subscribe(v => console.log(v));

## windowToggle

----
实例方法

将输入流分割成嵌套的流，分割时以 openings 发出值为开始，以 closingSelector 发出值为结束。

* <font face="仿宋">_类似于bufferToggle，但发出的是流而不是数组。_</font>

    -a--b---c---d---e---f----g----h----|-->

    --w-----------w-------------w------|--->

        -----|-->

                windowToggle

    -------------------------------------|-->
      \b---c|       \e---f|       \h|

当openings发出值时开始分离输入流上的数据，当closingSelector发出值时结分离行为，分离出的流作为输出流的值发出。

参数

| Name            | Type                           | Attribute | Description                                                         |
| --------------- | :----------------------------: | :-------: | ------------------------------------------------------------------: |
| openings        | Observable<O>                  |           | 开始新的分离的通知流                                                |
| closingSelector | function(value: O): Observable |           | 接收opeings的值作为参数，返回新的流，返回的流发出任意通知时结束分离 |

返回值

Observable<Observable<T>> 发出分割后的流的流。

示例

    Rx.Observable.fromEvent(document, 'click')
        .windowToggle(
            Rx.Obserbable.interval(1000),
            i => i%2 ? Rx.Observable.interval(500) : Rx.Observable.empty()
        )
        .mergeAll()
        .subscribe(v => console.log(v));

## windowWhen

----
实例方法

使用一个分割函数把输入流分割成多条子流。

* <font face="仿宋">_类似于bufferWhen，但发出的是流而不是数组。_</font>

    --a-----b----c---d---e--f----g----h---|--->

    ---------------|-->

                windowWhen

    --------------------------------------|--->
    \a------b----c| \d---e--f----g----h|

每当分割流发出通知时结束当前的分离行为，然后重新开启一个新的分割流。输出流被订阅后第一个分割流会立即被开启。

参数

| Name            | Type                   | Attribute | Description                                                                    |
| --------------- | :--------------------: | :-------: | -----------------------------------------------------------------------------: |
| closingSelector | function(): Observable |           | 不接收任何参数，返回新的流，返回的流发出任意通知时结束分离，同时开启下一个分离 |

返回值

Observable<Observable<T>> 输出分割流的流

示例

    Rx.Observable.fromEvent(document, 'click')
        .windowWhen(() => Rx.Observable.interval(1000 + Math.randow() * 4000))
        .map(win => win.take(2))
        .mergeAll()
        .subscribe(v => console.log(v));

----

    Rx.Observale.timer(0, 1000)
        .windowWhen(() => Rx.Observable.inerval(5000))
        .dow(() => console.log('NEW WINDOW'))
        .mergeAll()
        .subscribe(v => console.log(v));