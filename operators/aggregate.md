# 聚合类操作符

## count

----
实例方法

统计流上发出的值的数量，当流结束时发出最后的统计结果。

* <font face="仿宋">_统计输入流上一共发出了多少值。_</font>

        ------a-----------b----------c------|-->

                    count

        ------------------------------------3|-->

此操作符将输入流转换成一个发出输入流的值的统计数量的输出流。如果输入流上发出错误，输出流将会传递这个错误而不会发出正确的统计结果。如果输入流上一直没有发出结束通知，输出流将始终不会发射值。这个操作符可以接受一个可选的判定函数，在这个种情况下，输出流的结果是输入流上判定结果为true的值的个数。

| Name      | Type                                                          | Attribute | Description                                                                                                                                                                |
| --------- | :-----------------------------------------------------------: | :-------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| predicate | function(value: T, i: number, source: Observable<T>): Boolean | 可选      | 判定函数，用于判定输入流的值是否应该加入统计数量，为true时计入，反之不计入。这个判定函数接受3个参数，value：输入流上的值。index：输入流上值的索引，以0开始，source：输入流 |

## max

----
实例方法

此操作符操作一个发出数字（或者可以使用提供的函数进行比较的值）的流，当源 Observable 完成时输出流上的最大值。

    ----------23---------2---------3-------34-------|-->

                min

    ------------------------------------------------34|>

参数

| Name     | Type     | Attribute | Description                |
| -------- | :------: | :-------: | -------------------------: |
| comparer | Function | 可选      | 比较函数，用来比较流上的值 |

返回值

    Observable 输出最小值的流。

示例

    Rx.Observable.of(5, 4, 7, 2, 8)
        .min()
        .subscribe(x => console.log(x));

    interface Person {
        age: number;
        name: string;
    }

    Observable.of(
       {age: 7, name: 'Foo'},
       {age: 5, name: 'Bar'},
       {age: 9, name: 'Beer'}
    )
    .min((a, b) => a.age < b.age ? -1 : 1)
    .subscribe(x => console.log(x.name));

## min

----
实例方法

此操作符操作一个发出数字（或者可以使用提供的函数进行比较的值）的流，当源 Observable 完成时输出流上的最小值。

    ----------23---------2---------3-------34-------|-->

                min

    ------------------------------------------------2|>

参数

| Name     | Type     | Attribute | Description                |
| -------- | :------: | :-------: | -------------------------: |
| comparer | Function | 可选      | 比较函数，用来比较流上的值 |

返回值

    Observable 输出最小值的流。

示例

    Rx.Observable.of(5, 4, 7, 2, 8)
        .min()
        .subscribe(x => console.log(x));

    interface Person {
        age: number;
        name: string;
    }

    Observable.of(
       {age: 7, name: 'Foo'},
       {age: 5, name: 'Bar'},
       {age: 9, name: 'Beer'}
    )
    .min((a, b) => a.age < b.age ? -1 : 1>)
    .subscribe(x => console.log(x.name));

## reduce

----
实例方法

在源 Observable 上应用一个累积函数，当源 Observable 结束时把累积的值推送给输出流，可以接受一个初始的累积值。

* <font face="仿宋">_把源 Observable 上发出的所有值累积起来，使用一个累积函数来累积这些值。_</font>

        --------1-------2-------3---|-->

                reduce((acc,cur) => acc + cur, 0);

        ----------------------------4|

这个操作符的行为与数组的reduce方法相同。使用一个累积函数来处理流上的每一值，前一次累积后的值将作为下一次累积的初始值，把通过累积得到的最终值推送给输出流。注意，reduce操作符只会在源 Observable 发出结束通知后发出一个值，这相当于在使用 last 操作符后又使用了 scan 操作符。

源 Observable 上发出的值都会被累积函数处理。如果提供了初始值，这个值将成为整个累积过程的初始值，如果没有提供的话，从源 Observable 发出的第一个值就是整个累积过程的初始值。

参数

| Name        | Type                                      | Attribute | Description |
| ----------- | :---------------------------------------: | :-------: | ----------: |
| accumulator | function<R,T>(acc:R,cur:T,index:number):R |           | 累积函数    |
| seed        | R                                         | 可选。    | 初始值      |

返回值

Observable 把源 Observable 上的值都经过累积函数计算后得到的值作为值发送的流。

示例

    var clicksInFiveSeconds = Rx.Observable.fromEvent(document, 'click')
                                .takeUntil(Rx.Observable.interval(5000));
    var ones = clicksInFiveSeconds.mapTo(1);
    var seed = 0;
    var count = ones.reduce((acc,cur) => acc + cur, seed);

    count.subscribe(v => console.log(v));
