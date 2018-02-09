# 多播类操作符

## cache

## multicast

## publish

## publishBehavior

## publishLast

## publishReplay

## share

----
实例方法

输出流上的值可以共享给多个观察者。至少有一个观察者开始订阅这个条流时，它才会发射值，当所有的观察者都取消订阅时，它才会取消对输入源的订阅。因为它可同时把值同时发送给多个观察者，所以输出流可以认为是‘热’的。

输出流的行为类似于使用了.publish().refCount()的流，但不同的是后者不会重新订阅输入流。例如，Rx.Observable.of('test').publish().refCount() 在有新的观察者加入进不会将 'test' 重新发送给观察者，这与Rx.Observable.of('test').share()的行为是不同的。

    -----1---2-----3---4------5---|->

                share

    =====1===2=====3===4======5===|=>

返回值

Observable 在连接上时向观察者发送数据的流。

示例

    const source =  Rx.Observable.timer(1000)
        .do(() => console.log('sid effect'))
        .mapTo('result')

    // 以下两个订阅会触发do操作符的逻辑执行2次

    const sub1 = source.subscribe(v => console.log(v));

    const sub2 = source.subscribe(v => console.log(v));

    // 以下两个订阅只会将do操作符的逻辑触发一次

    const hot = source.share();

    const sub3 = hot.subscribe(v => console.log(v));

    const sub4 = hot.subscribe(v => console.log(v));
