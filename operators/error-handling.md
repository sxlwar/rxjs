# 错误处理操作符

## catch

----
实例方法

使用输出流来捕获源 Observable 上的错误或者抛出一个异常。

    ---------a--------b---------X------->

                                --------1----2----3-|>

                catch

    ---------a--------b-----------------1----2----3-|>

| Name     | Type     | Attribute | Description                                                                                                                                               |
| -------- | :------: | :-------: | --------------------------------------------------------------------------------------------------------------------------------------------------------: |
| selector | function |           | 错误处理函数，传入它的参数就是从源 Observable 上捕获到的异常，某些情况下你可能想重新执行源 Observable，那么这个函数返回的流将继续接着源 Observable 执行。 |

返回值

Observable 一个起源于源 Observable 或者从selector 函数中返回出来的流。

示例

    Rx.Observable.of(1,2,3,4,5)
        .map(x => {
            if( x === 4) {
                throw 'four!';
            } else {
                return x;
            }
        })
        .catch(err => Rx.Observable.of('I', 'II', 'III', 'IV', 'V'))
        .subscribe(x => console.log(x));

## retry

----
实例方法

返回一个源 Observable 的镜像流，但是镜像流上不包括源 Observable 上的错误。如果源 Observable 上发出了错误通知，这个操作符将重新尝试订阅源 Observable，尝试的次数不能超过参数指定的次数，而不会把错误通知发送出去。

    --1---2---3---X->

                retry(2);

    --1---2---3-----1---2---3------1---2---3--X->

任何来自源 Observable 的值都会在输出流上发出，即使在这期间发生了错误。例如，如果输出流在第一次发射2个值后发生了错误，这个时候它会重新尝试第二次发射，如果第二次成功，那么输出流上的值将是第一次发射的2个值，加上第二次发射的这些值

| Name  | Type   | Attribute | Description          |
| ----- | :----: | :-------: | -------------------: |
| count | number |           | 失败后继续尝试的次数 |

返回值

Observable 使用retry逻辑包装过的源 Observable 的镜像流。

示例

    Rx.Observable.interval(1000)
        .map(x => {
            const v = Math.random();

            if(v > 0.9) {
                throw 'dangerous';
            } else {
                return v;
            }
        })
        .retry(2)
        .subscribe(v => console.log(v));

## retryWhen

----
实例方法

返回一个源 Observable 的镜像流，但是镜像流上不包括源 Observable 上的错误。如果源 Observable 上抛出了错误，这个操作符会抛出引起错误的通知给通知流。如果通知流发出了错误通知或完成通知，则触发输出流的错误或完成通知，否则该方法会尝试重新订阅源 Observable。

    --------------------r---------------------r-------|----->

    -1--2--X->

                retryWhen

    -1--2---------------1--2------------------1--2---|-->

| Name     | Type                                    | Attribute | Description                                                               |
| -------- | :-------------------------------------: | :-------: | ------------------------------------------------------------------------: |
| notifier | function(error: Observable): Observable |           | 接收一通知流，根据此流的通知来决定什么时间重新执行或停止订阅源 Observable |

返回值

Observable 使用retry逻辑包装过的源 Observable 的镜像流。

示例

    Rx.Observable.interval(1000)
        .map(v => {
            if(v > 5) {
                throw v; // 由retryWhen接收此错误。
            } else {
                return v;
            }
        })
        .retryWhen(error => error.do(v => console.log(`value ${v} is too high!`))
                .delayWhen(v => Rx.Observable.timer(v * 1000))
        )
        .subscribe(v => console.log(v));
----
    Rx.Observable.interval(1000)
        .map(v => {
            if( v > 3) {
                throw v;
            } else {
                return v;
            }
        })
        .retryWhen(error => error.zip(Rx.Observable.range(1,4))
            .mergeMap(([err,i]) => {
                if(i > 3) {
                    return Rx.Observable.throw(err);
                } else {
                    console.log(`wait ${i} second before retry`);
                    return Rx.Observable.timer(i * 1000);
                }
            })
        )
        .catch(err => Rx.Observable.of('oh, it was give up to retry'))
        .subscribe(v => console.log(v));
