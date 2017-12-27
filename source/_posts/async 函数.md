---
title: async 函数
data: 2017-12-13
type: /code
---
## 1、含义
async 是什么？它就是Generator函数的语法糖。

``` javascript
const fs = require('fs');
const readFile = function (fileName) {
    return new Promise(function (resolve, reject) {
        fs.readFile(fileName, function(error, data) {
            if(error) return reject(error);
            resolve(data);
        });
    });
}

const gen = function *() {
    const f1 = yield.readFile('./etc/fstab');
    const f2 = yield.readFile('./etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
}
```