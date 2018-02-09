# 聚合类操作符

## count

## max

## min

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
