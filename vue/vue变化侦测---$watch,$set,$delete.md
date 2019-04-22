# $watch
先看一个vue的例子
```js
this.$watch('a.b', function(oldVal, newVal) {
    // xxx
}, {
    deep: true
});
this.$watch(function() {
    return this.a.b
}, function(oldVal, newVal) {
    // xxx
}, {
    deep: true
})
```
从上面可以看出，$watch支持三个参数：第一个为需要监听的属性（可以为字符串，可以为函数），第二个为回调，第三个为可用选项（deep：深度观测）。接下来就来实现这个功能。
```js
class MVVM {
    constructor() {

    }
    $watch(expOrFn, cb, options) {
        return new Watcher(this, expOrFn, cb, options)
    }
}
class Watcher {
    constructor(target, expOrFn, cb, options) {
        /** 这里对expOrFn是函数或者字符串做做一个区分，
        *   this.getter最终是一个函数
        * 调用this.getter后，就可以得到value
        */
        if (typeof expOrFn === 'function') {
            this.getter = expOrFn
        } else {
            this.getter = parsePath(expOrFn)
        }
        /**
        * 判断deep，如果deep为true，就需要在Dep.target = null之前，将target的子级属性设为响应式，并将这个watcher搜集到其dep中，这其中就可能出现多次搜集依赖的情况，故要想办法避免
        */
        if (options) {
            this.deep = !!options.deep
        } else {
            this.deep = false
        }
        // depIds就是用来避免重复订阅
        this.deps = []
        this.depIds = new Set()
        this.value = this.get()
    }
    get() {
        Dep.target = this;
        // xxxx
        let value = this.getter.call(this)
        /**
         * 如果是深度订阅，就要在Dep.target = null之前，将value子级的属性get一遍，让他们的dep订阅这个watcher，如此以来，子级属性变化，也会通知这个watcher
        */
        if (this.deep) {
            traverse(value)
        }
        Dep.target = null;
        return value
    }
    addDeps(dep) {
        const id = dep.id
        if (!this.depIds.has(id)) {
            this.deps.push(dep)
            this.depIds.add(id)
            dep.addSubs(this)
        }
    }
}
var uuid = 0
class Dep() {
    constructor() {
        this.id = uuid++
        this.subs = []
    }
    addSubs(watcher) {
        this.subs.push(watcher)
    }
    depend() {
        if (Dep.target) {
            Dep.target.addDeps(this)
        }
    }
}
function parsePath(exp) {
    return function() {
        let value = this
        exp.split('.').forEach(i => {
            value = value[i]
        })
        return value
    }
}
```
```js
function traverse(value) {
    let seenObject = new Set() // 用于处理重复订阅
    _traverse(value, seenObject)
    seenObject.clear() // 清空
}
function _traverse(value, seen) {
    const isA = Array.isArray(value)
    if (!isA && typeof value !== 'object') {
        return
    }
    if (value.__ob__) {
        /**
         * 这一步就是为了避免重复订阅
        */
        const depId = value.__ob__.dep.id
        if (seen.has(depId)) {
            return
        }
        seen.add(depId)
    }
    if (isA) {
        let i = value.length
        while(i--) {
            // 递归订阅
            _traverse(value[i], seen)
        }
    } else {
        const ary = Object.keys(value)
        let k = ary.length
        while(k--) {
            // 递归订阅
            // 这儿会去执行取值操作getter，然后对应的dep就回去收集当前的Dep.target
            _traverse(value[ary[k]], seen)
        }
    }
}
```

# $set
在vue中，修改数组中某个元素（this.ary[2]='aaa'），这样的书写方式，vue是没有办法去通知视图更新的
```js
class MVVM {
    ...
    $set(target, key, val) {
        if (Array.isArray(target)) {
            target.length = Math.max(target.length, key)
            // 使用splice，因为数组方法已经被拦截，所以可以通知依赖更新
            target.splice(key, 1, val)
            return val
        }
        if (key in target) {
            target[key] = val
            return val
        }
        // 新增一个新的属性,属性值就需要去建立响应式
        const ob = target.__ob__
        if (!ob) {
            // 如果target本身不是响应式数据，那么就不需要将值建立响应式
            target[key] = val
            return val
        }
        ob.defineReactive(ob.value, key, val)
        // 通知更新
        ob.dep.notify()
        return val
    }
}
```

# $delete
```js
class MVVM {
    ...
    $delete(target, key) {
        if (Array.isArray(target)) {
            // 使用splice，因为数组方法已经被拦截，所以可以通知依赖更新
            target.splice(key, 1)
            return 
        }
        const ob = target.__ob__
        if (!this._hasOwn(target, key)) {
            return
        }
        delete target[key]
        if (!ob) {
            return
        }
        // 通知更新
        ob.dep.notify()
        return
    }
    _hasOwn(target, key) {
        return target.hasOwnProperty(key)
    }
}
```