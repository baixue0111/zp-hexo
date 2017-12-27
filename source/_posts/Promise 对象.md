---
title: Promise 对象
data: 2017-10-26
type: /code
---
## Promise对象的含义
Promise是异步编程的一种解决方案，比传统的解决方案--回调函数和事件--更合理和强大。它是由社区最早提出和发现的。所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以使用同样的方法进行处理。

**Promise对象的2个特点：**
（1）对象的状态不受外界影响。**Promise**对象代表一个异步操作，有三种状态：**pending**(进行中)、**fulfilled**(已成功)和**rejected**(已失败)。只有异步操作的结果，可以选择当前是哪一种状态，任何其他操作都无法改变这个状态。
（2）一旦状态改变，就不会在变，任何时候都可以得到这个结果。**Promise**对象的状态改变，只有两种可能：从**pending**变为**fulfilled**和从**pending**变为**rejected**。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为**rejected**（已定型）。如果改变已经发生，在对**Promise**对象添加回调函数，也会理解得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

**Promise缺点：**
（1）无法取消Promise，一旦新建它就会立即执行，无法中途取消。
（2）如果不设置回调函数，promise内部抛出的错误，不会反应到外部。
（3）当处于**pending**状态时，无法得知目前进展到哪一个阶段（刚刚开始，还是即将完成）

## 基本用法
ES6规定，**Promise**对象是一个构造函数，用来生成**Promise**实例。

下面代码创建了一个**Promise**实例：

``` javascript
const promise = new Promise(function (resolve, reject) {
  
    if(异步操作成功) {
        resolve(value);
    } else {
        reject(error);
    }
});
```
**Promise**构造函数接受一个函数作为参数，该函数的两个参数分别是： **resolve**和**reject**。它们是两个函数，由JavaScript引擎提供，不用自己部署。
**resolve**函数的作用是，将**promise**对象的状态从“未完成”变为“成功”，在异步操作成功时调用，并将异步操作的结果，作为参数参数传递出去；**reject**函数的作用，将**Promise**对象的状态从“未完成”变为“失败”，在异步操作失败时调用，并将异步操作报出的错误作为参数传递出去。

**Promise**实例生成之后，可以用**then**方法分别指定**resolved**状态和**rejected**状态的回调函数。

``` javascript
promise.then(function (value) {
    // success
}, function (value) {
    // failure
})

```

**then**方法可以接受两个回调函数作为参数。第一个回调函数是**Promise**对象的状态变为**redolved**时调用的，第二个回调函数是**Promise**对象的状态变为**rejected**时调用的。其中，第二个函数是可选的，不一定要提供，这两个函数都接受**Promise**对象传出去的值作为参数。

下面是**Promise**对象简单例子：

``` javascript
function timeout(ms) {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, ms, 'done');
    });
}

timeout(100).then((value) => {
    console.log(value);
});

```
上面的代码中，**timeout**方法返回一个**Promise**实例，表示一段时间以后才会发生的结果。过了指定的时间（ms参数）以后，**Promise**实例的状态变为**resolved**，就会触发**then**方法绑定的回调函数。

**Promise**新建后就会立即执行。
``` javascript
let promise = new Promise(function (resolve, reject) {
    console.log('Promise');
    resolve();
});

promise.then(function () {
    console.log('resolved');
});

console.log('Hi!');

// Promise
// Hi!
// resolved

```
上面代码中，Promise新建后立即执行，所以首先输出的是**Promise**。然后，**then**方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以**resolved**最后输出。

下面是异步加载图片的例子：

``` html
<img src="" width="300" height="300" id="image">
```

``` javascript
function loadImageAsync(url) {
    return new Promise(function (resolve, reject) {
        const image = document.getElementById("image");

        image.onload = function () {
            resolve(image);
        };

        image.onerror = function () {
            reject(new Error('Could not load image at ' + url));
        };

        image.src = url;
    })
}

loadImageAsync('http://img.ishuo.cn/doc/1511/659-151123143T3.jpg');

```

上面的代码中，使用了**Promise**包装了一个图片加载的异步操作。如果加载成功，就调用**resolve**方法，否则就调用**reject**方法。

下面是一个用**Promise**对象实现的Ajax操作的例子。

``` javascript
const getJson = function (url) {
    const promise = new Promise(function (resolve, reject) {
        const handler = function () {
            if(this.readyState !== 4) {
                return;
            }
            if(this.status === 200) {
                resolve(this.response);
            } else {
                reject(new Error(this.statusText)); 
            }
        }
        const client = new XMLHttpRequest();
        client.open("GET", url);
        // 请求状态改变时
        client.onreadystatechange = hanlder;
        client.responseType = "json";
        client.setRequestHeader("Accept", "application/json");
        client.send();
    });

    return promise;
}

getJson('/posts.json').then(function (json) {
    console.log('Contents: ' + json);
}, function (error) {
    console.error('出错了', error);
})

```
上面代码中，**getJson**是对**XMLHttpRequest**对象的封装，用于发出一个针对**JSON**数据的**HTTP**请求，并返回一个**Promise**对象。需要注意的是：在**getJson**内部，**resolve**函数和**reject**函数调用时，都带有参数。

如果调用**resolve**函数和**reject**函数时都带有参数，那么它们的参数会被传递给回调函数。
**reject**函数的参数通常是**Error**对象的实例，表示抛出错误；**resolve**函数的参数除了正常的值之外，还可能是另一个Promise实例， 比如：

``` javascript
const p1 = new Promise(function (resolve, reject) {
    // ...
})

const p2 = new Promise(function (resolve, reject) {
    // ...
    resolve(p1);
})
```

上面的代码，p1和p2都是Promise实例，但是p2的**resolve**方法将p1作为参数，这时p1的状态决定着p2的状态。如果p1的状态是 **pending**，那么p2就会等待p1的状态改变，如果p1的状态已经是**resolved**或是**rejected**，那么p2的回调就会立即执行。

## Promise.prototype.then()
**Promise**实例具有**then**方法，也就是说，**then**方法是定义在原型对象**Promise.prototype**上的。它的作用是为**promise**实例添加状态改变时的回调函数。
**then**方法返回的是一个新的**promise**实例，（注意，不是原来的那个**promise**实例）。因此可以采用链式写法，即**then**后面在调用另一个**then**方法。

``` javascript
getJson("/posts.json").then(function (json) {
    return json.post;
}).then(function (post) {
    // ...
})

```

上面的代码使用**then**方法，依次指定了两个回调函数。第一个回调函数完成以后，会将结果作为参数，传入第二个回调函数。
采用链式的**then**，可以指定一组依照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个**promise**对象（即有异步操作）。这时，后一个回调函数，就会等待该**promise**对象的状态发生变化，才会调用。

``` javascript
getJson("/post/1.json").then(function (post) {
    return getJson(post.commentURL);
}).then(function funA(comments) {
    console.log("resolved: ", comments);
}, function funB(error) {
    console.log("rejected: ", error);
});
``` 

上面代码中，第一个**then**方法指定的回调函数，返回的是另一个**promise**对象。这时，第二个**then**方法指定的是回调函数，就会等待这个新的**promise**对象状态发生改变。如果变为**resolved**，就调用**funA**，如果状态变为**rejected**，就调用**funB**

## Promise.prototype.catch()
