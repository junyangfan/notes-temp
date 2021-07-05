// 手写Promise
class YX {
    // 定义静态属性(状态)
    static PENDING = 'pending'
    static FULFILLED = 'fulfilled'
    static REJECTED = 'rejected'
    // 初始化 executor(执行者，相当于callback)
    constructor(executor) {
        // 初始化
        this.status = YX.PENDING
        // 存放成功或者失败的值
        this.value = null
        // 存放异步任务
        this.callbacks = []
        // 处理异常状态
        try {
            // 执行((resolve, reject) => {})
            executor(this.resolve.bind(this), this.reject.bind(this))
        } catch (error) {
            // 发生错误就是拒绝态
            this.reject(error)
        }
    }
    // 成功态
    resolve(value) {
        // 如果状态已经改变，则不能再次改变状态
        if(this.status === YX.PENDING) {
            this.status = YX.FULFILLED
            this.value = value
            // 变为异步任务
            setTimeout(() => {
                // 处理等待态成功的异步任务
                this.callbacks.map(callback => {
                    callback.onFulfilled(value)
                })
            })
        }
    }
    // 拒绝态
    reject(reason) {
        // 如果状态已经改变，则不能再次改变状态
        if(this.status === YX.PENDING) {
            this.status = YX.REJECTED
            this.value = reason
            // 变为异步任务
            setTimeout(() => {
                // 处理等待态拒绝的异步任务
                this.callbacks.map(callback => {
                    callback.onRejected(reason)
                })
            })
        }
    }
    // then
    then(onFulfilled, onRejected) {
        // 如果不想要成功态，只想要失败态，那么传个null也行，
        // 例如：new Promise((resolve, reject) => {}).then(null, reject => {})
        // 所以自己封装个函数给他返回
        if (typeof onFulfilled !== 'function') {
            onFulfilled = () => {
                // then穿透处理，例如:new Promise((reslove) => {reslove('nh')}).then().then(v => console.log(v))
                return this.value
            }
        }
        if (typeof onRejected !== 'function') {
            // then穿透处理
            onRejected = () => this.value
        }
        // 支持链式调用then(递归)
        const promise = new YX((resolve, reject) => {
            // 如果状态为成功态，则执行传递的函数，并把值带过去
            if (this.status === YX.FULFILLED) {
                // 放到异步队列里
                setTimeout(() => {
                    this.parse(promise, onFulfilled(this.value), resolve, reject)
                })
            }
            // 如果状态为失败态，则执行传递的函数，并把值带过去
            if (this.status === YX.REJECTED) {
                // 放到异步队列里
                setTimeout(() => {
                    this.parse(promise, onRejected(this.value), resolve, reject)
                })
            }
            // 如果是等待态，则把等会要处理的函数放到callbacks中，等到时间了，自动处理
            // 例如：
            // new Promise((reslove, reject) => {
            //     setTimeout(() => {
            //         reslove('解决')
            //     }, 1000)
            // }).then(v => {
            //     console.log(v)
            // })
            if (this.status === YX.PENDING) {
                this.callbacks.push({
                    onFulfilled: value => {
                        this.parse(promise, onFulfilled(value), resolve, reject)
                    },
                    onRejected: reason => {
                        this.parse(promise, onRejected(reason), resolve, reject)
                    },
                })
            }
        })
        return promise
    }
    // 代码优化
    parse(promise, result, resolve, reject) {
        // 不允许循环调用
        if (promise === result) {
            throw new TypeError('Chaining cycle detected')
        }
        try {
            if (result instanceof YX) {
                // 如果返回的值是个Promise，则进行处理(再次调用then即可)
                result.then(resolve, reject)
            } else {
                // 如果返回的是普通值，直接返回
                resolve(result)
            }
        } catch (error) {
            reject(error)
        }
    }
    // 静态方法resolve
    static resolve(value) {
        return new YX((resolve, reject) => {
            if (value instanceof YX) {
                value.then(resolve, reject)
            } else {
                resolve(value)
            }
        })
    }
    // 静态方法reject
    static reject(value) {
        return new YX((resolve, reject) => {
            reject(value)
        })
    }
    // 静态方法all
    static all(promises) {
        return new YX((resolve, reject) => {
            const values = []
            promises.forEach(promise => {
                promise.then(value => {
                    values.push(value)
                    // 如果都成功，则改变状态
                    if (values.length === promises.length) {
                        resolve(values)
                    }
                }, reason => {
                    // 如果有一个失败，则走reject
                    reject(reason)
                })
            })
        })
    }
    // 静态方法race
    static race(promises) {
        return new YX((resolve, reject) => {
            promises.map(promise => {
                promise.then(value => {
                    // 如果有一个成功，则直接改变状态即可
                    resolve(value)
                }, reason => {
                    reject(reason)
                })
            })
        })
    }
}


// new Promise((resolve, reject) => {})


/**
 * 未优化代码,未完成整体功能

class YX {
    // 定义静态属性(状态)
    static PENDING = 'pending'
    static FULFILLED = 'fulfilled'
    static REJECTED = 'rejected'
    // 初始化 executor(执行者，相当于callback)
    constructor(executor) {
        // 初始化
        this.status = YX.PENDING
        // 存放成功或者失败的值
        this.value = null
        // 存放异步任务
        this.callbacks = []
        // 处理异常状态
        try {
            // 执行((resolve, reject) => {})
            executor(this.resolve.bind(this), this.reject.bind(this))
        } catch (error) {
            // 发生错误就是拒绝态
            this.reject(error)
        }
    }
    // 成功态
    resolve(value) {
        // 如果状态已经改变，则不能再次改变状态
        if(this.status === YX.PENDING) {
            this.status = YX.FULFILLED
            this.value = value
            // 变为异步任务
            setTimeout(() => {
                // 处理等待态成功的异步任务
                this.callbacks.map(callback => {
                    callback.onFulfilled(value)
                })
            })
        }
    }
    // 拒绝态
    reject(reason) {
        // 如果状态已经改变，则不能再次改变状态
        if(this.status === YX.PENDING) {
            this.status = YX.REJECTED
            this.value = reason
            // 变为异步任务
            setTimeout(() => {
                // 处理等待态拒绝的异步任务
                this.callbacks.map(callback => {
                    callback.onRejected(reason)
                })
            })
        }
    }
    // then
    then(onFulfilled, onRejected) {
        // 如果不想要成功态，只想要失败态，那么传个null也行，
        // 例如：new Promise((resolve, reject) => {}).then(null, reject => {})
        // 所以自己封装个函数给他返回
        if (typeof onFulfilled !== 'function') {
            onFulfilled = () => {
                // then穿透处理，例如:new Promise((reslove) => {reslove('nh')}).then().then(v => console.log(v))
                return this.value
            }
        }
        if (typeof onRejected !== 'function') {
            // then穿透处理
            onRejected = () => this.value
        }
        // 支持链式调用then(递归)
        return new YX((resolve, reject) => {
            // 如果状态为成功态，则执行传递的函数，并把值带过去
            if (this.status === YX.FULFILLED) {
                // 放到异步队列里
                setTimeout(() => {
                    // 处理异常，返回失败态
                    try {
                        // 链式调用处理
                        const result = onFulfilled(this.value)
                        
                        if (result instanceof YX) {
                            // 如果返回的值是个Promise，则进行处理
                            // result.then(value => {
                            //     resolve(value)
                            // }, reason => {
                            //     reject(reason)
                            // })
                            result.then(resolve, reject)
                        } else {
                            // 如果返回的是普通值，直接返回
                            resolve(result)
                        }
                    } catch (error) {
                        reject(error)
                    }
                })
            }
            // 如果状态为失败态，则执行传递的函数，并把值带过去
            if (this.status === YX.REJECTED) {
                // 放到异步队列里
                setTimeout(() => {
                    // 处理异常，返回失败态
                    try {
                        const result = onRejected(this.value)
                        if (result instanceof YX) {
                            // 如果返回的值是个Promise，则进行处理
                            result.then(resolve, reject)
                        } else {
                            // 如果返回的是普通值，直接返回
                            resolve(result)
                        }
                    } catch (error) {
                        reject(error)
                    }
                })
            }
            // 如果是等待态，则把等会要处理的函数放到callbacks中，等到时间了，自动处理
            // 例如：
            // new Promise((reslove, reject) => {
            //     setTimeout(() => {
            //         reslove('解决')
            //     }, 1000)
            // }).then(v => {
            //     console.log(v)
            // })
            if (this.status === YX.PENDING) {
                this.callbacks.push({
                    onFulfilled: value => {
                        // 处理异步成功态的错误
                        try {
                            const result = onFulfilled(value)
                            if (result instanceof YX) {
                                // 如果返回的值是个Promise，则进行处理
                                result.then(resolve, reject)
                            } else {
                                // 如果返回的是普通值，直接返回
                                resolve(result)
                            }
                        } catch (error) {
                            reject(error)
                        }
                    },
                    onRejected: reason => {
                        // 处理异步拒绝态的错误
                        try {
                            const result = onRejected(reason)
                            if (result instanceof YX) {
                                // 如果返回的值是个Promise，则进行处理
                                result.then(resolve, reject)
                            } else {
                                // 如果返回的是普通值，直接返回
                                resolve(result)
                            }
                        } catch (error) {
                            reject(error)
                        }
                    },
                })
            }
        })
    }
}

 */