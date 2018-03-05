# 条件操作符

## defaultIfEmpty

----
实例方法

如果源 Observable 上没有发出过任何有效值，则使用传入这个操作符的参数作为输出流的有效值。

* <font face="仿宋">_源 Observable 上没有发出有效值就结束了的话，使用这个值作为输出流的默认值。_</font>

        --------------------------|>

                defaultIfEmpty(48);

        --------------------------48|>

如果源 Observable 没有发出过有效值，则使用这个值作为默认值，反之，使用源 Observable 上发出的值。

| Name         | Type  | Attribute          | Description                          |
| ------------ | :---: | :----------------: | -----------------------------------: |
| defaultValue | any   | 可选。默认值：null | 当源 Observable 是空的时候的默认值。 |

返回值

Observable 一个要么发出源 Observable 上的值，要么发出一个默认值的流。

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var clicksBeforeFive = clicks.takeUntil(Rx.Observable.interval(5000));
    var result = clicksBeforeFive.defaultIfEmpty('no clicks');

    result.subscribe(x => console.log(x));

## every

----
实例方法

创建一个发出布尔值的流，true表示源 Observable 的值都可以通过判定函数的检测，false 表示某些值无法通过判定函数检测。

    -----1---------2------3-------4------5----|--->

                every(x => x < 10);

    ------------------------------------------true|>

返回值

Observable

示例

    Rx.Observable.of(1,2,3,4,5)
        .every(x => x < 6)
        .subscribe(x => console.log(x));

## find

----
实例方法

发射出从流上找到的第一个符合给定条件的值。

* <font face="仿宋">_找到第一个符条件的值，将它发送给输出流。_</font>

        ----2--------10-----------4-----------25--------|-->

                find(x => x%5 ===0);

        -------------10|>

从源 Observable 上查找第一个通过判定函数的值，将它发送给输出流。和first操作符不同的是，它必须接收一个判定函数，而且在没有找到有效值的情况下不会发出错误通知。

| Name         | Type  | Attribute          | Description                          |
| ------------ | :---: | :----------------: | -----------------------------------: |
| predicate | function<T>(value: T, index: number, source: Observable<T>):boolean   | | 判定函数 |
| thisArg | any | 可选 | 用来决定判定函数内部的this指向 |

返回值

Observable 发出源 Observable 上能过判定函数检测的第一个值的流。

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var result = clicks.find(ev => ev.target.tagName === 'DIV');

    result.subscribe(v => console.log(v));

## findIndex

----
实例方法

发出源 Observable 上第一个通过判定函数检测的值的索引。

* <font face="仿宋">_各find操作符的行为类似，只不过它发出的不是值，而是值的索引。_</font>

        ----2--------10-----------4-----------25--------|-->

                find(x => x%5 ===0);

        -------------2|>

从源 Observable 上查找第一个通过判定函数的值，将它的索引发送给输出流。和first操作符不同的是，它必须接收一个判定函数，而且在没有找到有效值的情况下不会发出错误通知。

| Name         | Type  | Attribute          | Description                          |
| ------------ | :---: | :----------------: | -----------------------------------: |
| predicate | function<T>(value: T, index: number, source: Observable<T>):boolean   | | 判定函数 |
| thisArg | any | 可选 | 用来决定判定函数内部的this指向 |

返回值

Observable 发出源 Observable 上能过判定函数检测的第一个值的索引的流。

示例

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var result = clicks.findIndex(ev => ev.target.tagName === 'DIV');

    result.subscribe(v => console.log(v));

## isEmpty

----
实例方法

如果源 Observable 没有发出过任何有效值，输出流发出true，反之，发出false。

    --------------------------------|->

                isEmpty

    --------------------------------true|>

返回值

    Observable 一个发出 true 或 false 的流。

示例

    var source = Rx.Observable.empty();

    source.isEmpty()
        .subscribe(v => console.log(v));
