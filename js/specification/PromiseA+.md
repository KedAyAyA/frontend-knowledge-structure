##  术语
  1.1 ```promise```是一个带有表现符合规范的```then```方法的对象或者函数

  1.2 ```thenable```是一个定义```then```方法的对象或者函数

  1.3 ```value```是一个任意的合法的JavaScript值，包括```undefined, thenable, promise``` 

  1.4 ```exception```是一个使用```throw```语句抛出的异常

  1.5 ```reason```是一个解释为什么```promise```被```rejected```的值

##  需求
  ### 2.1 Promise 状态

  一个Promise必须属于三种状态之一：pending, fulfilled, rejected  
  - 2.1.1 当处于```pending```
  
    - 2.1.1.1 可能转换为```fulfilled```或```rejected```状态

  - 2.1.2 当处于```fulfilled```

    - 2.1.2.1 不能转换为其他状态
    - 2.1.2.2 一定有一个不可改变的value

  - 2.1.3 当处于```rejected```

    - 2.1.3.1 不能转换为其他状态
    - 2.1.3.2 一定有一个不可改变的reason

  ### 2.2 ```then```方法

  一个promise必须提供一个```then```方法来获得他当前或最终的value或者reason

  一个promise的```then```方法接受两个参数

  ``` promise.then(onFulfilled, onRejected) ```

  - 2.2.1 ```OnFulfilled```和```onRejected```都是可选的参数

    - 2.2.1.1 如果```onFulfilled```不是一个函数，他必须被忽略
    - 2.2.1.2 如果```onRejected```不是一个函数，他必须被忽略

  - 2.2.2 如果```onFulfilled```是一个函数

    - 2.2.2.1 必须在```promise```状态变为```fulfilled```后调用，以```promise```的value作为第一个参数

    - 2.2.2.2 不可在```promise```状态变为```fulfilled```之前调用

    - 2.2.2.3 不可多次调用

  - 2.2.3 如果```onRejected```是一个函数

    - 2.2.3.1 必须在```promise```状态变为```rejected```后调用，以```promise```的reason作为第一个参数

    - 2.2.3.2 不可在```promise```状态变为```rejected```之前调用

    - 2.2.3.3 不可多次调用

  - 2.2.4 ```onFulfilled```或```onRejected```直到执行上下文栈内仅包含平台代码时必须执行 [3.1]

  - 2.2.5 ```onFulfilled```或```onRejected```必须作为函数被调用（不含有this）[3.2]

  - 2.2.6 ```then```可能在同一个promise里被调用多次

    - 2.2.6.1 如果```promise```处于```fulfilled```的状态，所有各自的```onFulfilled```回调方法必须按照他们原始调用的顺序依次调用

    - 2.2.6.2 如果```promise```处于```onRejected```的状态，所有各自的```onRejected```回调方法必须按照他们原始调用的顺序依次调用

  - 2.2.7 ```then```必须返回一个promise[3.3]
    ```
    promise2 = promise1.then(onFulfilled, onRejected)
    ```
    - 2.2.7.1 如果```onFulfilled```或```onRejected```返回了一个value ```x```,运行promise解决程序, [[Resolve]](promise2, x)

    - 2.2.7.2 如果```onFulfilled```或```onRejected```抛出了一个exception ```e```, ```promise2```必须以e作为reason rejected

    - 2.2.7.3 如果```onFulfilled```不是一个函数并且```promise1```处于fulfilled状态，```promise2```必须处于fulfilled状态并且与promise1具有相同的value

    - 2.2.7.4  如果```onRejected```不是一个函数并且```promise1```处于rejected状态，```promise2```必须处于rejected状态并且与promise1具有相同的reason

  ### 2.3 Promise解决程序
  promise解决程序是一个以```promise```和```value```作为输入的抽象操作，我们表示为```[[Resolve]](promise, x)```。如果x是```thenable```，该方法尝试使```promise```采取```x```的状态，在```x```表现得至少有点像一个promise的假设下。否则，使```promise```变为值为x的fulfilled状态

  运行```[[Resolve]](promise, x)```，按下列步骤表现

  - 2.3.1 如果```promise```和```x```是相同的object, 使用```TypeError```作为reason reject ```promise```

  - 2.3.2 如果 ```x```是promise, 采纳他的状态[3.4]

    - 2.3.2.1 如果 ```x```处于pending, ```promise```必须维持```pending```状态直到```x```被fulfilled或rejected

    - 2.3.2.2 如果```x```处于fulfilled状态，用相同的 value fulfill ```promise```

    - 2.3.2.3 如果```x```处于rejected状态，用相同的reason reject ```promise```

  - 2.3.3 如果 ```x```是对象或者函数

    - 2.3.3.1 使```then```成为```x.then```[3.5]
    - 2.3.3.2 如果检索到```x.then```导致了一个异常```e```,将```e```当做reason reject promise

    - 2.3.3.3 如果```then```是一个函数，将```x```作为```this```调用函数，第一个参数为```resolvePromise```,第二个参数为```rejectPromise```

      - 2.3.3.3.1 如果```resolvePromise```以参数value ```y```调用 运行[[Resolve]](promise, y)

      - 2.3.3.3.2 如果```rejectPromise```以参数reason ```r```调用，使用r作为reason reject promise

      - 2.3.3.3.3 如果```resolvePromise```和```rejectPromise```被调用，或者以相同参数被调用多次，仅第一次调用优先，其他则被忽略

      - 2.3.3.3.4 如果调用```then```并抛出异常e
      
        - 2.3.3.3.4.1 如果```resolvePromise```和```rejectPromise```已经被调用，使用value x fulfill promise

        - 2.3.3.3.4.1 否则，使用reason e reject promise 

    - 2.3.3.4 如果```then```不是一个函数，使用value x fulfill promise

  - 2.3.4 如果```x```不是函数和对象，使用value x fulfill promise