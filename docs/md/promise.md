<iframe src="/article/html/promise.html" style="width: 100%; height: 100px; border:0;"></iframe>

测试代码 test1 & test2 : 
``` js
var app = {};
function test1(){
    app.test1 = new Promise(function(resolve, reject){
        setTimeout(function(){
            resolve(123);
        }, 1000)
    }).then(function(data){
        console.log(data,1);
        return 2;
    }, function(error){
        console.log(error,2);
        return false;
    }).then(function(data){
        console.log(data,3);
        return new Promise(function(resolve, reject){
            setTimeout(function(){
                resolve('new');
            }, 3000)
        }).then(function(data){
            console.log('new then', data);
            return 124;
        })
    }).catch(function(err){
        console.log(err);
    });
    console.log(app.test1);
}

function test2() {
    var end = false;
    function test(type){
        var res;
        var timeid;
        var timeids = [];
        if (type === 0) {
            var queue = [];
            for (var i =1; i < 20; i++) {
                (function(i){
                    queue.push(new Promise((resolve, reject) => {
                        timeids[i] = setInterval(function() {
                            if (end) {
                                clearInterval(timeids[i]);
                                resolve(i*100);
                            }
                        }, 1000);
                    }));
                })(i);
            }
            return Promise.all(queue);
        } else {
            return new Promise((resolve, reject) => {
                timeid = setInterval(function() {
                    if (end) {
                        clearInterval(timeid);
                        resolve(0);
                    }
                }, 1000);
            });
        }
    }
    app.test2 = test(0);
    setTimeout(() => {
        end = true;
        console.log(app.test2);
    }, 3000);
}
document.querySelector("#test-1").addEventListener("click", test1);
document.querySelector("#test-2").addEventListener("click", test2);
console.log(app);
```

promise: 
``` js
(function() {
    function Promise(executor){
        var self = this;
        set(self, 'pending');
        // 执行同步代码, 绑定 resolve 和 reject 参数的方法
        try{
            executor(function(){
                self.resolve.apply(self, arguments);
            }, function(){
                self.reject.apply(self, arguments);
            });
        } catch(error) {
            // 执行出错?
            set(self, 'rejected', error);
        }
        return this;
    }
    
    Promise.prototype = {
        constructor: Promise,
        queue: [],
        resolve: function (value){
            set(this, 'resolved', value);
            return next(this);
        },
        reject: function (error){
            set(this, 'rejectd', error);
            return next(this);
        },
        then: function(onFulfilled, onRejected){
            var self = this;
            // 如果状态是pending, 将后面的then全部放入队列
            if(this['[[PromiseStatus]]'] == 'pending'){
                this.queue.push([this, arguments]);
                return this;
            }
            // 如果状态是 resolved
            if (this['[[PromiseStatus]]'] == 'resolved') {
                // 如果存在onFulfilled方法
                if (typeof onFulfilled === 'function') {
                    // 尝试执行onFulfilled, 并将结果赋值给[[PromiseValue]]
                    try{
                        this['[[PromiseValue]]'] = onFulfilled.call(this, this['[[PromiseValue]]']);
                    } catch (error) {
                        // (暂时不执行这个条件语句的true) 
                        // 调用onRejected, 如果返回值为true, 那么当前错误可以视为已解决, 不为true, 那么会忽略后面的then, 直到遇到catch
                        this['[[PromiseValue]]'] = error;
                        if (0 && onRejected.call(this, error)) {
                            this['[[PromiseValue]]'] = undefined;
                        } else {
                            // 执行出错, 设置状态为rejectd
                            set(this, 'rejectd', error);
                        }
                    }
                    // (暂时不执行这个条件语句的true) 
                    if (0 && this['[[PromiseStatus]]'] !== 'rejectd'){
                        set(this, 'resolved', this['[[PromiseValue]]']);
                    }
                }
                // 如果返回值是 promise, 那么直接返回这个promise
                if (this['[[PromiseValue]]'] instanceof Promise) {
                    return this['[[PromiseValue]]'].then(function(data){ 
                        set(self, 'resolved', data);
                        return data;
                    }).catch(function(err) {
                        set(self, 'rejected', err);
                        return err;
                    });
                }
            // 如果状态是rejectd 跳过, 直到遇到catch
            } else if (this['[[PromiseStatus]]'] == 'rejectd') {
                return this;
            }
            // 执行下一个then
            return next(this);
        },
        catch: function(func){
            // 有错误的时候, 才会调用 func
            if (this['[[PromiseStatus]]'] === 'rejectd'){
                if (typeof func === 'function' && func.call(this, this['[[PromiseValue]]'])) {
                    set(this, 'rejectd', undefined);
                }
            }
            return this;
        }
    }
    Promise.all = function(queue){
        var list = [];
        return new Promise(function(resolve, reject) {
            if (Array.isArray(queue)) {
                var count = queue.length;
                for (var i = 0; i < queue.length; i++) {
                    if (queue[i] instanceof Promise){
                        (function(i){
                            queue[i].then(function(data){
                                count--;
                                list[i] = data;
                                if (!count) {
                                    resolve(list);
                                }
                                return data;
                            });
                        })(i);
                    } else {
                        list[i] = queue[i];
                        if (!count) {
                            resolve(list);
                        }
                    }
                }
            } else {
                reject(new Error('queue is not an Array'))
            }
        })
    }
    // 设置promise的属性
    function set(that, status, value){
        that['[[PromiseStatus]]'] = status;
        that['[[PromiseValue]]'] = value;
        return that;
    }
    // 调用下一个
    function next(promise){
        // 如果队列里有元素
        if (promise.queue.length){
            // 取出元素
            var args = promise.queue.shift();
            // 执行
            return promise.then.apply(args[0], args[1]);
        }
        return promise;
    }
    window.Promise = Promise;
})();
```
