- #### 目标
> 我们要实现这样的功能？
1. Promise有三个状态：pending, fulfilled,rejected。还有一个值value。
2. Promise状态的改变只有两种方式：pending -> fulfilled 或者 pending -> rejected。
3. Promise状态改变后将不能再被改变。
4. 只能通过new Promise的方式生成Promise对象。
5. Promise构造函数必须接受一个函数fn。
6. fn必须接受两个参数（这两个参数必须是函数）：resolve和reject。
7. Promise对象必须要有then方法。
8. then方法会接受两个参数（两个参数必须是函数）：onFulfilled和onRejected。
9. then必须返回一个新的Promise。
10. resolve的执行会调用then方法中的onFulfilled方法；reject的执行会调用then方法中的onRejected方法。

- #### 基于上面的目标，现在我们一步步的实现对应的功能

1. 定义构造函数Promise，定义状态state。
```
function Promise(){
    this.state = 'pending';
}
```
2. 只能通过new的方式生成Promise。
```
function Promise(){
    if (!(this instanceof Promise))
        throw new TypeError('Promise是构造函数，必须使用new的方式！');

    this.state = 'pending';
}
```
3. 构造函数必须接受一个函数，并且函数有两个也是函数类型的参数。获取value或者reason，并赋值给Promise。
```
function Promise(fn){
    if (!(this instanceof Promise))
        throw new TypeError('Promise是构造函数，必须使用new的方式');

     if (!(fn instanceof Function))
        throw new TypeError('构造函数Promise必须接受一个函数作为参数');
        
    this.state = 'pending';
    this.value = null;

    fn((value)=>{
        this.value = value;
        resolve(this);
    }, (reason)=>{
        this.value = reason;
        reject(this);
    });
}

function resolve(self){

}

function reject(self){

}
```
4. 执行resolve或者reject后，状态需要改变。且改变后，将不能再被改变。
```
function resolve(self){
    if(self.state !== 'pending')
        return;
    
    self.state = 'fulfilled';
}

function reject(self){
    if(self.state !== 'pending')
        return;
    
    self.state = 'rejected';
}
```
5. 给Promise定一个then方法，并接受两个参数。
```
Promise.prototype.then = function(onFulfilled, onRejected){}
```
6. 为了能够链式的调用Promise，我们需要在then方法中返回一个新的Promise。
```
Promise.prototype.then = function (onFulfilled, onRejected) {
    let promise = new Promise(() => { });

    return promise;
}
```
7. 为了能正确的执行then中的回调方法，我们需要给将then中的回调方法绑定到对应的Promise对象上。
```
function Promise(fn) {
    if (!(this instanceof Promise))
        throw new TypeError('Promise是构造函数，必须使用new的方式');

    if (!(fn instanceof Function))
        throw new TypeError('构造函数Promise必须接受一个函数作为参数');

    this.state = 'pending';
    this.value = null;
    this.onFulfilled = null;  //定义onFulfilled方法，为了能够执行对应Promise下的回调
    this.onRejected = null;
    //定义onRejected方法，为了能够执行对应Promise下的回调

    fn((value) => {
        this.value = value;
        resolve(this);
    }, (reason) => {
        this.value = reason;
        reject(this);
    });
}

Promise.prototype.then = function (onFulfilled, onRejected) {
    let promise = new Promise(() => { });

    this.onFulfilled = onFulfilled;  //将对应的方法挂载到对应的Promise上
    this.onRejected = onRejected;
    
    return promise;
}
```
8. 为了能够依次的执行then中的回调，我们需要递归的调用对应的方法。
```
function resolve(self) {
    if (self.state !== 'pending')
        return;
    //递归的调用每个Promise的onFulfilled回调
    let value;
    if (self.onFulfilled && self.value) {
        self.state = 'fulfilled';
        value = self.onFulfilled(self.value);

        if (self.promise) {
            self.promise.value = value;
            resolve(self.promise);
        }
    }
}

function reject(self) {
    if (self.state !== 'pending')
        return;

    //递归的调用每个Promise的onFulfilled回调
    let value;
    if (self.onRejected && self.value) {
        self.state = 'rejected';
        value = self.onRejected(self.value);

        if (self.promise) {
            self.promise.value = value;
            reject(self.promise);
        }
    }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
    let promise = new Promise(() => { });

    this.onFulfilled = onFulfilled;  //将对应的方法挂载到对应的Promise上
    this.onRejected = onRejected;  //将对应的方法挂载到对应的Promise上
   
    resolve(this);
    reject(this);

    this.promise = promise;

    return promise;
}
```
9. then永远要先于resolve或者reject执行。
> 比如如下的代码：
```
new Promise((resolve, reject)=>{
    resolve(1);
}).then().then().then()
```
> 上面代码正确的执行顺序为：生成Promise对象 =》然后立即执行构造函数中的参数fn => 然后执行后续的所有then方法 =》最后再执行resolve(1)

> 所以给resolve或者reject增加延迟即可
```
function Promise(fn) {
    if (!(this instanceof Promise))
        throw new TypeError('Promise是构造函数，必须使用new的方式');

    if (!(fn instanceof Function))
        throw new TypeError('构造函数Promise必须接受一个函数作为参数');

    this.state = 'pending';
    this.value = null;
    this.onFulfilled = null;  //定义onFulfilled方法，为了能够执行对应Promise下的回调
    this.onRejected = null;  //定义onRejected方法，为了能够执行对应Promise下的回调

    fn((value) => {
        setTimeout(() => {
            this.value = value;
            resolve(this);
        }, 0);
    }, (reason) => {
        setTimeout(() => {
            this.value = reason;
            reject(this);
        }, 0);
    });
}
```
> [这里是源代码，可以去查看](https://github.com/Greency/xPromise)