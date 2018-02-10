# 工具类操作符

## do

----
实例方法

在输入流发出值是执行一个有副作用的操作，但输出流的值和输入流是完全相同的。

* <font fact="仿宋">_在输入流有值发出时执行一个函数，只要函数运行时没有发生错误输出的值仍然和输入流上的值一样。_</font>

    ---1--------2---------3----|-->

                do(v => console.log(v));

    ---1--------2---------3----|-->

生成一个输入流的镜像，但是这个镜像在发出值之前会执行一个具有副作用的操作，输入流上的所有通知都能安全的传递到输出流上。这个方法非常适合用来调试代码。

注意，这个操作符和subscribe并不一样，它的返回值也是一个流，所有在没有订阅之前，这些副作用的操作是永远不会执行的，并不像subscribe一样会触发流的执行。

参数

| Name           | Type     | Attribute | Description                |
| -------------- | :------: | :-------: | -------------------------: |
| nextOrObserver | Observer | function  | 可选                       |  |
| error          | function | 可选      | 处理输入流上错误通知的函数 |
| complete       | function | 可选      | 处理输入流上结束通知的函数 |

返回值

Observable 会在发出值之前执行副作用的输入流镜像。

示例

    Rx.Observable.fromEvent(document, 'click')
        .do(v => console.log(v))
        .map(v => v.clientX)
        .subscribe(v => console.log(v));

## delay

----
实例方法

指定一个延迟时间，当输入流有上值到达时，输出流将延迟发射这个值，也可以指定一个日期对象，在到达此日期对象指定的值时发射值。

* <font face="仿宋">_以指定的延迟时间发送值。_</font>

    --------a------------b------------|->

                delay

    --------------a------------b------|>

如果此操作符的参数是一个数字，将被当成一个毫秒数，在输入流上的值到达时，输出流将延迟毫秒数指定的时间发送值，也就是说输出流发射值时最低要保持这个延迟时间。

如果提供的参数是一个日期对象，这个流将会在到达这个日期时才开始发射值，这个日期之前在输入流上产生的值不会被发射。

参数

| Name      | Type           | Attribute | Description                                  |
| --------- | :------------: | :-------: | -------------------------------------------: |
| delay     | number 或 Date |           | 延迟发射值的时间间隔或者开始发射值的日期对象 |
| scheduler | Scheduler      | 可选      | 有来提供时间概念的调度器                     |

返回值

Observable 以指定是延迟时间或日期规则发送值。

示例

    Rx.Observable.fromEvent(document,'click')
        .delay(2000)
        .subscribe(v => console.log(v));
----
    Rx.Observable.fromEvent(document,'click')
        .delay(new Date(2018-05-01))
        .subscribe(v => console.log(v));

## delayWhen

## dematerialize

## finally

## let

## materialize

## observeOn

## subscribeOn

## timeInterval

## timestamp

## timeout

## timeoutWith

## toArray

## toPromise
