# 多播类操作符

## multicast

----
实例方法

输出流可以将输入流共享给多个订阅

    ----1----2-----3----4-----5----|-->

        multicast(() => new Subject());

    ====1====2=====3====4=====5====|-->

multicast可以在2种模式下工作。

第一种方式是可以给这种操作符提供一个Subject或都一个可以返回Subject的工厂函数。这种情况下，将会得到一个特殊的Observable——ConnectableObservable。和普通的Observable一样这个流可以被多次订阅，但是此时并不会触发对输入流的订阅。只有当调用了ConnectableObservable的connet方法时，才会触发对输入流的订阅，这意味关你可以手动控制何时对输入流开始订阅。这个订阅可以在多个订阅者中共享，这意味者一个发起ajax的请求可以只发出一次，即使多个订阅者的订阅都会触发些请求。

使用ConnectableObservable的通常情况是在第一个订阅发生时就调用它的connect方法，此后在有其它订阅都订阅时都使用这个订阅，在最后一个订阅者取消订阅后再取消。为了避免这种复杂的逻辑，ConnectableObservable提供了一个特殊的操作符refCount，当调用这个方法时，它返回一个Observable，它会统计自己的订阅者数量，只要有订阅者存在，它都会使ConnectableObservable处于订阅状态，如果你不需要手动的决定何时开始连接及何时断开连，可以使用这个方法。

第二种方式是给此操作符添加传递一个可选的第二个参数——selector function。这个函数接收输入流的镜像流，返回另一个流，返回的流将被当作其它操作符的输入流。在这种模式下，multicast的第一个参数不直接传入一个Subject，而是必须传入一个产生Subject的工厂函数。在这种模式下，multicast返回的仅仅是一个普通的Observable，而不是一个ConnectableObservable，因此对输出流的多次订阅会触发的实际是多次订阅。然而，如果在selector函数内部多次订阅输入流时只会触发一次对输入流的订阅。因此，这种模式适用那些需要多次订阅输出流，而只想订阅一次输入流时特别适用。

给multicast提供的Subject被当作输入流的代理，也就是说输入流的值都会通过这个Subject传递给订阅者。因此，如果Subject上存在一些特殊属性时，由multicast返回的Observable也同样具有这些属性。如果仅给这个操作符提供一个Subject，包括Subject的其它子类，例如：AsyncSubject，BehaviorSubject等，可以使用publish，publishLast等操作符替代。这些都是multicast的包装，在其内部都会硬编码一个Subject。

参数

| Name                    | Type                | Attribute | Description                                                                                            |
| ----------------------- | :-----------------: | :-------: | -----------------------------------------------------------------------------------------------------: |
| subjectOrSubjectFactory | Function 或 Subject |           | 用来创建Subject的工厂函数或者需要推入值的Subject                                                       |
| selector                | Function            | 可选      | 在需要时可以多次调用输入流，但是不会多次订阅输入流。输入流的订阅者将在订阅期间收到输入流上的所有通知。 |

返回值

Observable 可以把输入流的值共享给多个订阅者的流。

示例

    const seconds = Rx.Observable.interval(1000)

    const connectableSeconds = seconds.multicast(new Subject());

    connectableSeconds.subscribe(v => console.log(`first: ${v}`));
    connectableSeconds.subscribe(v => console.log(`second: ${v}`));


    connectableSeconds.connect();

----

    const seconds= Rx.Observable.interval(1000);

    seconds
        .multicast(
            () => new Subject(),
            seconds => seconds.zip(seconds)
        )
        .subscribe();

## publish

----
实例方法

返回一个ConnectableObservable，也就是订阅行为只有调用输出流的connect方法时才真正的发生。

    -------1-----2------------3------4--------5----|-->

                publish

    =======1=====2============3======4========5====|==>

参数

| Name     | Type     | Attribute | Description                                                                                            |
| -------- | :------: | :-------: | -----------------------------------------------------------------------------------------------------: |
| selector | Function | 可选      | 在需要时可以多次调用输入流，但是不会多次订阅输入流。输入流的订阅者将在订阅期间收到输入流上的所有通知。 |

返回值

ConnectableObservable

示例

    const source = Rx.Observable.interval(1000);

    const example = source.do(v => console.log('Do something'))
        .publish();

    const sub1 = example.subscribe(v => console.log(`first subscribe: ${v}));

    const sub2 = example.subscribe(v => console.log(`second subscribe: ${v}));

    setTimeout(() => {
        example.connect();
    },5000);

## publishBehavior

----
实例方法

返回值

ConnectableObservable

## publishLast

----
实例方法

返回值

ConnectableObservable

## publishReplay

----
实例方法

返回值

ConnectableObservable

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
