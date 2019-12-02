## 认识RxJS

1. 异步的处理方案有callback,promise,async/await，但是随着应用需求的越来越复杂，编写异步代码依然很困难。常见的问题有：
    
    * 竞态条件（对同一资源进行异步存取时，请求顺序会导致请求的结果不同）
    * 记忆体泄漏（在做SPA网站时，在A页面监听body的scroll事件，但是页面切换以后没有把事件移除掉）
    * 复杂的状态
    * 异常处理（try/catch可以捕获同步行为的异常，但是异步的错误就难以捕获）
2. RxJS将函数式编程和响应式编程结合。

    * 函数式编程就是用函数来思考我们的问题以及编写代码
    * 响应式编程则是当资源发生变动的时候，由资源自动告诉我发生了变化

## 函数式编程（Functional Programming）

1. 函数式编程的核心是通过函数（function）来思考问题，但不是所有的语言都支持函数式编程，支持FP的语言至少符合函数为一等公民的特性，也就是说函数可以被赋值给变量，也可以被当做参数传递给函数，也可以作为函数的返回值。
2. 纯函数（Pure Function）是指一个function给予相同的参数，永远会返回相同的值，并且没有任何显著的副作用。
3. 函数式编程的优势：

    * 可读性更高
    * 因为纯函数的特性，执行结果不会依赖外部状态而且不会改变外部状态，使函数式编程更容易除错和编写单元测试
    * 函数式编程更容易做并行处理，因为只做运算不处理I/O

## Observable

1. 建立Observable
    ```javascript
    // 通常我们创建可观察对象的方式是from,of,fromEvent,fromPromise;create是最基本的方法
    var observable = Rx.Observable
        .create(function(observer) {
            observer.next('Jerry');
            observer.next('Anna');
        });
    console.log('start');
    // 訂閱這個 observable	
    observable.subscribe(function(value) {
        console.log(value);
    });
    console.log('end');
    ```
    上面代码是同步的，打印结果为：
    ```
    start
    Jerry
    Anna
    end
    ```
    observable处理异步操作
    ```javascript
    // 通常我们创建可观察对象的方式是from,of,fromEvent,fromPromise;create是最基本的方法
    var observable = Rx.Observable
        .create(function(observer) {
            observer.next('Jerry');
            observer.next('Anna');
            setTimeout(()=>{
                observable.next('RxJS');
            });
        });
    console.log('start');
    // 訂閱這個 observable	
    observable.subscribe(function(value) {
        console.log(value);
    });
    console.log('end');
    ```
    异步打印结果为：
    ```
    start 
    Jerry
    Anna
    RxJS
    ```
1. 观察者（Observer）

    可观察对象（Observable）可以被订阅，订阅可观察对象的叫观察者。观察者会有三个方法：next()，error(),complete()。
    ```javascript
    var observable = Rx.Observable
        .create(function(observer) {
            observer.next('Jerry');
            observer.next('Anna');
        });
    var observer = {
        next: function(value) {
            console.log(value);
        },
        error: function(error) {
            console.log(error);
        },
        complete: function(){
            console.log('complete');
        }
    }
    observable.subscribe(observer);
    ```

2. 创建可观察对象
    
    * of

        ```javascript
        var observable = Rx.Observable.of('Jerry','Anna'); // Jerry,Anna
        ```

    * from

        ```javascript
        var observable = Rx.observable.from([1,2,3]); // 1,2,3
        var observable = Rx.observable.from('Jerry'); // J,e,r,r,y
        var observable = Rx.observable.from(new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve('Hello');
            },1000);
        })); // Hello
        ```

    * fromEvent

    ```javascript
    var observable = Rx.observable.fromEvent(document.body,'click'); // MouseEvent{...}
    ```
    * empty,never,throw
    ```javascript
    var source = Rx.Observable.empty(); // empty 没有做任何事情
    var source = Rx.Observable.never(); // never 永远不做任何事情
    var source = Rx.Observable.throw('error'); // throw 抛出错误
    source.subscribe({
        next: function(value) {
            console.log(value)
        },
        complete: function() {
            console.log('complete!');
        },
        error: function(error) {
            console.log(error);
        }
    });
    // complete!
    //
    // Throw Error:error
    ```

    * interval

    ```javascript
    var source = Rx.Observable.interval(1000); // 每隔一秒发送一个从零开始递增的整数
    // 0
    // 1
    // 2
    ```

    * timer

    ```javascript
    var source = Rx.Observable.timer(1000, 5000); // 第一个参数代表发送第一个值之前的等待时间，第二个参数代表第一个值发送之后的等待时间。所以结果是1秒之后发送0，然后每隔5秒发送一次
    // 0
    // 1
    // 2
    var source = Rx.Observable.timer(1000); // 只有一个参数的时候，结果为等待一秒发送0，然后结束
    // 0 
    ```

1. Subscription

    在订阅观察者对象后，会返回一个订阅对象（Subscription）,这个对象具有释放资源的方法（unsubscribe）。
    ```javascript
    var source = Rx.Observable.timer(1000, 1000);
    // 取得 subscription
    var subscription = source.subscribe({
        next: function(value) {
            console.log(value)
        },
        complete: function() {
            console.log('complete!');
        },
        error: function(error) {
        console.log('Throw Error: ' + error)
        }
    });

    setTimeout(() => {
        subscription.unsubscribe() // 停止订阅
    }, 5000);
    // 0
    // 1
    // 2
    // 3
    // 4
    ```

## 操作符Operators

+ take:取前几个元素就结束
    ```javascript
    var source = Rx.Observable.interval(1000);
    var example = source.take(3);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 1
    // 2
    // complete
    ```

+ first:行为和take(1)相同

+ takeUntil:接受一个可观察对象，当这个可观察对象被订阅时，原来的可观察对象会直接进入完成状态
    ```javascript
    var source = Rx.Observable.interval(1000);
    var click = Rx.Observable.fromEvent(document.body, 'click');
    var example = source.takeUntil(click);     
    
    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 1
    // 2
    // 3
    // complete (点击了body)
    ```

+ concatAll:将由可观察对象组成的可观察对象展平
    ```javascript
    var obs1 = Rx.Observable.interval(1000).take(5);
    var obs2 = Rx.Observable.interval(500).take(2);
    var obs3 = Rx.Observable.interval(2000).take(1);

    var source = Rx.Observable.of(obs1, obs2, obs3);

    var example = source.concatAll();

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 1
    // 2
    // 3
    // 4
    // 0
    // 1
    // 0
    // complete
    ```

+ skip:忽略前几个元素
    ```javascript
    var source = Rx.Observable.interval(1000);
    var example = source.skip(3);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 3
    // 4
    // 5
    ```

+  takeLast:取最后两个元素
    ```javascript
    var source = Rx.Observable.interval(1000).take(6);
    var example = Rx.Observable.takeLast(2);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ', err); },
        complete: () => { console.log('complete'); }
    });
    // 4
    // 5 
    // complete
    ```

+ last:与takeLast(1)效果一致

+ contact:可以把多个可观察对象合并成一个
    ```javascript
    var source = Rx.Observable.interval(1000).take(3);
    var source2 = Rx.Observable.of(3)
    var source3 = Rx.Observable.of(4,5,6)
    var example = source.concat(source2, source3);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 1
    // 2
    // 3
    // 4
    // 5
    // 6
    // complete
    ```

+ startWith:在可观察对象的最前面插入一个元素
    ```javascript
    var source = Rx.Observable.interval(1000);
    var example = source.startWith(0);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 0
    // 1
    // 2
    // 3
    ```

+ merge:合并可观察对象，与contact不同的是，merge是把多个可观察对象同时处理
    ```javascript
    var source = Rx.Observable.interval(500).take(3);
    var source2 = Rx.Observable.interval(300).take(6);
    var example = source.merge(source2);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 0
    // 1
    // 2
    // 1
    // 3
    // 2
    // 4
    // 5
    // complete
    ```
    宝石图(Marble Diagram)：
    ```
    source：----0----1----2|
    source2:--0--1--2--3--4--5|
    example:--0-01--21-3--(24)--5|
    ```

+ combineLatest:可以获取每个可观察对象最后发送的值，最后输出成一个值。可以接受多个可观察对象，最后一个参数为处理函数
    ```javascript
    var source = Rx.Observable.interval(500).take(3);
    var newest = Rx.Observable.interval(300).take(6);

    var example = source.combineLatest(newest, (x, y) => x + y);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 1
    // 2
    // 3
    // 4
    // 5
    // 6
    // 7
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source: ----0----1----2|
    newest: --0--1--2--3--4--5|
    example:----01--23-4--(56)--7|
    ``

+ zip:会将每个可观察对象相同位置的元素传入callback
    ```javascript
    var source = Rx.Observable.interval(500).take(3);
    var newest = Rx.Observable.interval(300).take(6);

    var example = source.zip(newest, (x, y) => x + y);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 2
    // 4
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source: ----0----1----2|
    newest: --0--1--2--3--4--5|
    example:----0----2----4|
    ```

+ withLastForm:只有在主要的可观察对象发送新的值时，才会执行回调函数
    ```javascript
    var main = Rx.Observable.from('hello').zip(Rx.Observable.interval(500), (x, y) => x);
    var some = Rx.Observable.from([0,1,0,0,0,1]).zip(Rx.Observable.interval(300), (x, y) => x);

    var example = main.withLatestFrom(some, (x, y) => {
        return y === 1 ? x.toUpperCase() : x;
    });

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    ```
    宝石图(Marble Diagram):
    ```
    main   :----h----e----l----l----o|
    some   :--0--1--0--0--0--1|
    example:----h----e----l----L----o|
    ```

+ scan:和Array.reduce方法类似
    ```javascript
    var source = Rx.Observable.from('hello')
             .zip(Rx.Observable.interval(600), (x, y) => x);

    var example = source.scan((origin, next) => origin + next, '');

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // h
    // he
    // hel
    // hell
    // hello
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source :-----h-----e-----l-----l-----o|
    example:-----h-----(he)-----(hel)-----(hell)-----(hello)|
    ```

+ buffer:一共有5个方法（buffer，bufferCount，bufferTime，bufferToggle，bufferWhen），常用的是前三个

    buffer: buffer要传入一个可观察对象(source2)，他会把原来的可观察对象(source)发送的值缓存在数组中，等到source2发送元素时，就会把缓存的数组发送出去
    ```javascript
    var source = Rx.Observable.interval(300);
    var source2 = Rx.Observable.interval(1000);
    var example = source.buffer(source2);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // [0,1,2]
    // [3,4,5]
    // [6,7,8]
    ```
    宝石图(Marble Diagram):
    ```
    source :--0--1--2--3--4--5--6--7--8--9
    source2:---------0---------1---------2
    example:---------([0,1,2])---------([3,4,5])---------([6,7,8])
    ```

    bufferTime: bufferTime和上面的例子类似
    ```javascript
    var source = Rx.Observable.interval(300);
    var example = source.bufferTime(1000);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // [0,1,2]
    // [3,4,5]
    // [6,7,8]
    ```

    bufferCount: 使用数量做缓存
    ```javascript
    var source = Rx.Observable.interval(300);
    var example = source.bufferCount(3);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // [0,1,2]
    // [3,4,5]
    // [6,7,8]
    ```

+ delay:延迟可观察对象开始发送的时间点

    ```javascript
    var source = Rx.Observable.interval(300).take(5);

    var example = source.delay(500);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 1
    // 2
    // 3
    // 4
    ```
    宝石图(Marble Diagram):
    ```
    source :--0--1--2--3--4|
    example:-------0--1--2--3--4|
    ```

+ debounce:和buffer，bufferTime一样，debounce和debounceTime一个是传入observable一个传入毫秒值，常用的是debounceTime。作用是每次接受新的元素后会等待一段时间，如果这段时间内没有接受新的元素则将元素发送；如果接受新的元素，则释放原来的元素并重新计时间。

    ```javascript
    var source = Rx.Observable.interval(300).take(5);
    var example = source.debounceTime(1000);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 4
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source :--0--1--2--3--4|
    example:--------------4|
    ```

+ throttle:有throttle和throttleTime两个方法，一个传入observable，另一个传入毫秒。作用是先发送接受的元素，然后等待一段时间，等这段时间过了之后再发送新的元素。

    ```js
    var source = Rx.Observable.interval(300).take(5);
    var example = source.throttleTime(1000);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // 0
    // 4
    // complete
    ```

+ distinct:过滤掉相同的值

    ```js
    var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'b'])
            .zip(Rx.Observable.interval(300), (x, y) => x);
    var example = source.distinct();

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // a
    // b
    // c
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source: --a--b--c--a--b|
    example:--a--b--c------|
    ```
    我们可以传入一个callback，回调函数会接受一个元素，并回传我们真正希望比对的值：
    ```js
    var source = Rx.Observable.from([{ value: 'a'}, { value: 'b' }, { value: 'c' }, { value: 'a' }, { value: 'c' }])
            .zip(Rx.Observable.interval(300), (x, y) => x);
    var example = source.distinct((x) => {
        return x.value
    });

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // {value: "a"}
    // {value: "b"}
    // {value: "c"}
    // complete
    ```
    事实上，distinct()会在背后创建一个Set，当接受到元素时会先去判断Set内是否有相同的值，如果有则不会发送，没有则会存到Set并发送。所以尽量不要直接把distinct用在一个无限的observable里，这样可能会使Set越来越大，建议使用第二个参数flushes或者用distinctUntilChanged
    ```js
    var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'c'])
                .zip(Rx.Observable.interval(300), (x, y) => x);
    var flushes = Rx.Observable.interval(1300);
    var example = source.distinct(null, flushes);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // a
    // b
    // c
    // c
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source: --a--b--c--a--c|
    flushes:------------0---...
    example:--a--b--c-----c|
    ```

+ distinctUntilChanged:distinctUntilChanged跟distinct一样会把相同的元素过滤掉，但是distinctUntilChanged只会和最后一个发送的元素比较，而不是每个都比较。
    ```js
    var source = Rx.Observable.from(['a', 'b', 'c', 'c', 'b'])
                .zip(Rx.Observable.interval(300), (x, y) => x);
    var example = source.distinctUntilChanged()

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    // a
    // b
    // c
    // b
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source: --a--b--c--c--b|
    example:--a--b--c-----b|
    ```

+ catch:在Rxjs中可以使用catch来处理错误，catch可以接受一个observable来发送新的值。
    ```js
    var source = Rx.Observable.from(['a','b','c','d',2])
                .zip(Rx.Observable.interval(500), (x,y) => x);

    var example = source
                .map(x => x.toUpperCase())
                .catch(error => Rx.Observable.of('h'));

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    }); 
    ```
    宝石图(Marble Diagram):
    ```
    source: ----a----b----c----d----2|
            map(x => x.toUpperCase())
            ----A----B----C----D----x|
            catch(error => Rx.Observable.of('h'))
    example:----A----B----C----D----h|
    ```
    我们也可以通过返回Rx.Observable.empty()来结束。catch回调函数里的第二个参数会接受当前的observable对象，我们也可以回传当前的observable对象来实现重新执行。

+ retry:当一个observable发生错误时，重新尝试可以使用retry()方法。retry可以传入一个数值，表示重新尝试的次数，错误会在使用完次数之后抛出。
    ```js
    var source = Rx.Observable.from(['a','b','c','d',2])
                .zip(Rx.Observable.interval(500), (x,y) => x);

    var example = source
                    .map(x => x.toUpperCase())
                    .retry(1);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    }); 
    // a
    // b
    // c
    // d
    // a
    // b
    // c
    // d
    // Error: TypeError: x.toUpperCase is not a function
    ```
    宝石图(Marble Diagram):
    ```
    source :----a----b----c----d----2|
            map(x => x.toUpperCase())
            ----a----b----c----d----X|
            retry(1)
    example:----a----b----c----d--------a----b----c----d----X|
    ```

+ repeat:实现一直重复订阅的效果。
    ```js
    var source = Rx.Observable.from(['a','b','c'])
            .zip(Rx.Observable.interval(500), (x,y) => x);

    var example = source.repeat(2);

    example.subscribe({
        next: (value) => { console.log(value); },
        error: (err) => { console.log('Error: ' + err); },
        complete: () => { console.log('complete'); }
    });
    
    // a
    // b
    // c
    // a
    // b
    // c
    // complete
    ```
    宝石图(Marble Diagram):
    ```
    source : ----a----b----c|
            repeat(2)
    example: ----a----b----c----a----b----c|
    ```
