##  前言

```Promise```作为ES6极为重要的一个特性，将我们从无限的回调地狱中解脱出来，变为链式的编写回调，大大提高的代码的可读性。

使用Promise是极为简单的，但只停留在会使用阶段还是会让我们不知不觉踩到一些坑的。本文会结合```Promise/A+```规范与示例来深入学习```Promise```

本文较长，例子很多，末尾有一些应用 ^_^

## Promise

- 兼容性

  ```Promise```作为是浏览器的内置函数对象，除IE不支持外所有主流版本的浏览器都是支持的，如不需支持IE可以在不引入polyfill的情况下放心食用

- 作用

  js是单线程的，异步是通过Event Loop来实现的，那么需要一种更加友好的方式来实现我们的异步操作，而不是无限的回调嵌套

  ```javascript
  // no promise
  callback(() => {
    callback(() => {
      callback(() => {
        ...
      })
    })
  })

  // with promise
  new Promise((resolve, reject) => {

  }).then(() => {

  }).then(() => {

  }).then(() => {

  })
  ```

- API
  
  先来简单看一下Promise提供的一些API(注意区分静态方法与实例方法)
  - 构造函数Promise

    Promise是一个构造函数，可以通过```new```操作符来创建一个Promise实例
    ```javascript
    let promise = new Promise((resolve, reject) => {
      resolve(1)
    })
    ```

  - 静态方法

    - Promise.resolve
    - Promise.reject
    - Promise.all
    - Promise.race

  - 实例方法

    - Promise.prototype.then
    - Promise.prototype.catch

- 结合规范与样例
  - 规范

    [官方英文原版规范](https://promisesaplus.com/#notes)

    tips: 
    1.  规范是规范，实现是实现，实现是按照规范来实现的，但不一定完全等同于规范。
    2.  本文仅限浏览器环境测试，node环境可能会不一致
    
  - 状态

    一个Promise实例只能处于```pending,fulfilled,rejected```三种状态中的一种。每次创建的promise实例都会处于```pending```状态，并且只能由```pending```变为```fulfilled```或```reject```状态。一旦处于```fulfilled```或```rejected```状态无论进行什么操作都不会再次变更状态(规范2.1)

    ```javascript
    // 创建一个处于pending状态的promise实例
    let promise1 = new Promise((resolve, reject) => {

    })

    // 调用resolve()会使promise从pending状态变为fulfilled状态
    let promise2 = new Promise((resolve, reject) => {
      resolve(1)
    })

    // 调用reject()会使promise从pending状态变为rejected状态
    let promise3 = new Promise((resolve, reject) => {
      reject(1)
    })

    // 无论如何更改resolve与reject的执行顺序，promise4始终只会处于先调用后转换的状态(状态一旦变为fulfilled或rejected后不会再次改变)
    let promise4 = new Promise((resolve, reject) => {
      resolve(1)
      reject(1)
    })
    ```

  - then

    - 参数
    
      实例方法```then```的回调函数接受两个参数，```onFulfilled```和```onRejected```，都为可选参数(规范2.2.1)

      ```javascript
      // onFulfilled会在状态变为fulfilled后调用，onFulfilled参数value为resolve的第一个参数，onRejected同理。(规范2.2.2与2.2.3)

      let promise = new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve(1)
        }, 1000) 
      })

      promise.then((value) => {
        console.log(value)  // 1秒后输出1
      }, (reason) => {

      })

      // 可被同一个promise多次调用(规范2.2.6)
      promise.then()

      promise.then((value) => {
        console.log(value) // 1秒后输出1
      })
      ```

    - 返回值

      ```then```方法必须返回一个```promise```实例(规范2.2.7)

      假定 ```promise2 = promise1.then(onFulfilled, onRejected)```

      ```javascript
      var a = new Promise((resolve, reject) => {
        resolve(1)
      })

      var b = new Promise((resolve, reject) => {
        reject(1)
      })

      // 如果```onFulfilled```或```onRejected```返回了一个value ```x```,运行promise解决程序(下一节), [[Resolve]](promise2, x) (规范2.2.7.1)
      var a1 = a.then(function onFulfilled(value) {
        return 1
      }, function onRejected (reason) {

      })

      var b1 = b.then(function onFulfilled(value) {
        
      }, function onRejected (reason) {
        return 1
      })

      // 如果```onFulfilled```或```onRejected```抛出了一个exception ```e```, ```promise2```必须以e作为reason rejected (规范2.2.7.2)
      // 故下方a2 b2 都为状态是rejected的promise实例
      var a2 = a.then(function onFulfilled(value) {
        throw Error('test')
      }, function onRejected (reason) {

      })

      var b2 = b.then(function onFulfilled(value) {
        
      }, function onRejected (reason) {
        throw Error('test')
      })

      // 如果promise1处于fulfilled状态并且onFulfilled不是一个函数，那么promise2会以与promise1具有相同value和相同的状态, 但是与promise1并不是同一Promise实例;若为rejected状态以此类推
      var a3 = a.then()
      a3 === a // false
      var b3 = b.then()
      b3 === b // false
      ```

  - 解决程序resolve

    在规范中, promise解决程序是一个以```promise```和```value```作为输入的抽象操作，我们表示为```[[Resolve]](promise, x)```。

    可以认为在实现里```Promise.resolve()```与```new Promise((resolve, reject) => {})```中的```resolve```都为解决程序。规范中x对应实现中resolve的第一个参数，规范中promise在实现中等价于当前promise

    可以理解为```Promise.resolve(x)```与```new Promise(resolve) => {resolve(x)}```等价


    ```javascript
    var c = new Promise((resolve, reject) => {
      resolve(1)
    })

    var d = new Promise((resolve, reject) => {
      reject(1)
    })

    var e = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('5s后resolve')
      }, 5000)
    })
    
    // 在返回值章节我们了解到，onFulfilled如果为函数在不抛出错误的情况下，会调用解决程序处理返回值x
    // 如果x是promise, 采纳他的状态(注意，但then的返回值并不与x相等)(规范2.3.2)
    var c1 = Promise.resolve().then(function onFulfilled() {
      return c
    })

    c1 === c // false

    // pending状态下的，只要e状态变为，e1也会改变(规范2.3.2.1) 
    var e1 = Promise.resolve().then(function onFulfilled () {
      return e
    })

    var c2 = Promise.resolve()
    // 如果promise和x是相同的对象, 使用TypeError作为reason reject promise(规范2.3.1)
    var c3 = c2.then(function onFulfilled () {
      return c3
    })
    ```
    概念：如果一个函数或者对象具有then属性，那么叫做thenable(例: { then: function(){} })

    ```javascript
    // 如果x是thenable 但x.then不是一个函数或者对象,直接返回状态为fulfilled，value为x的promise实例(规范2.3.3.4)
    var c4 = c1.then(function onFulfilled () {
      return {}
    })

    var c5 = c1.then(function onFulfilled () {
      return {
        then: 2
      }
    })

    // 如果x是thenable 并且x.then是一个函数,那么就会调用这个then方法执行，并且两个参数resolve与reject拥有改变返回的promise状态的权利，会按解决程序处理，如果不调用两个参数方法，则返回的promise为pending状态，值得一提的是，调用then方法的时候回以返回对象x作为then的this绑定调用(规范 2.3.3.3)

    var c6 = c1.then(function onFulfilled () {
      return {
        test: 'test',
        then: function (resolve, reject) {
          console.log(this.test)
          resolve('thenable')
        }
      }
    })

    var c7 = c1.then(function onFulfilled () {
      function x () {}
      x.test = 'test'
      x.then = function (resolve, reject) {
        console.log(this.test)
        resolve('thenable')
      }
      return x
    })

    var c8 = c1.then(function onFulfilled () {
      return {
        test: 'test',
        then: function (resolve, reject) {
          console.log(this.test)
          reject('thenable')
        }
      }
    })

    // 返回pending状态的promise
    var c9 = c1.then(function onFulfilled () {
      return {
        then: function () {}
      }
    })

    // 如果x不是对象也不是函数，则返回状态为fulfilled的promise，value为x

    var c10 = c1.then(function onFulfilled () {
      return 'hehe'
    })
    ```

  综上可以总结几点
    1.  ```new Promise, promise.then, Promise.resolve()```都会返回promise实例
    2.  Promise状态一旦改变不可再更改
    3.  ```then```方法返回对象的不同会导致不同的结果，如果意外返回了thenable有可能会造成意想不到的结果

## 应用

  - 封装一个异步请求

    ```javascript
    var promiseXHR = new Promise((resolve, reject) => {
      var xhr = new XMLHttpRequest()
      xhr.open('GET', 'http://www.baidu.com', true)
      xhr.onreadystatechange = function () {
        if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
          resolve(xhr.response)
        } else {
          reject()
        }
      }
      xhr.send(data)
    })
    ```

  - 同时请求按序处理

    ```javascript
    // 假设有5个章节的数据，我们需要分别获取并读取到c中，利用promise和es6数组api我们可以写出简洁优雅的代码完成这个需求

    var list = ['Chapter1', 'Chapter2', 'Chapter3', 'Chapter4', 'Chapter5']

    var getData = function (key) {
      return new Promise((resolve, reject) => {
        // 模拟一些返回时间不相同的异步操作
        setTimeout(() => {
          resolve(key)
        }, 4000 * Math.random())
      })
    }

    var c = ''

    list.map(i => getData(i))
        .reduce((accumulator, current) => {
          console.log(accumulator)
          return accumulator.then(() => {
            return current
          }).then(value => {
            c += value
          })
        }, Promise.resolve(''))
    ```

##  后记
  
  通过阅读规范并写demo进行验证测试，对Promise的理解更加深入了，也能更好的使用它了