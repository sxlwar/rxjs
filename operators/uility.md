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
