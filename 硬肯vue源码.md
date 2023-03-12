源码阅读可能会迟到，但是一定不会缺席！

众所周知，以下代码就是 vue 的一种直接上手方式。通过 cdn 可以在线打开 vue.js。一个文件，一万行源码，是万千开发者赖以生存的利器，它究竟做了什么？让人品味。

```xml
<html>
<head></head>
<body>
    <div id="app">
        {{ message }}
    </div>
</body>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
    var app = new Vue({
        el: '#app',
        data: {
            message: 'See Vue again!'
        },
    })
</script>
</html>
复制代码
```

源码cdn地址：[cdn.jsdelivr.net/npm/vue/dis…](https://link.juejin.cn?target=https%3A%2F%2Fcdn.jsdelivr.net%2Fnpm%2Fvue%2Fdist%2Fvue.js)，当下版本：v2.6.11。

本瓜选择生啃的原因是，可以更自主地选择代码段分轻重来阅读，一方面测试自己的掌握程度，一方面追求更直观的源码阅读。

当然你也可以选择在 [github.com/vuejs/vue/t…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Ftree%2Fdev%2Fsrc) 分模块的阅读，也可以看各路大神的归类整理。

其实由于本次任务量并不算小，为了能坚持下来，本瓜将源码尽量按 500 行作为一个模块来形成一个 md 文件记录（[分解版本共 24 篇感兴趣可移步](https://link.juejin.cn?target=https%3A%2F%2Ftuaran.github.io%2Fviews%2F2020%2FstudyVuesc1.html)），结合注释、自己的理解、以及附上对应查询链接来逐行细读源码，**此篇为合并版本**。

**目的：自我梳理，分享交流。**

最佳阅读方式推荐：先点赞👍再阅读📖，靴靴靴靴😁

## 正文

### 第 1 行至第 10 行

// init

```javascript
(
    function (global, factory) {
        typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() : typeof define === 'function' && define.amd ? define(factory) : (global = global || self, global.Vue = factory());
    }(
        this,
        function () {
            'use strict';
            //...核心代码...
        }
    )
);
// 变形
if (typeof exports === 'object' && typeof module !== 'undefined') { // 检查 CommonJS
    module.exports = factory()
} else {
    if (typeof define === 'function' && define.amd) { // AMD 异步模块定义 检查JavaScript依赖管理库 require.js 的存在 [link](https://stackoverflow.com/questions/30953589/what-is-typeof-define-function-defineamd-used-for)
        define(factory)
    } else {
        (global = global || self, global.Vue = factory());
    }
}
// 等价于
window.Vue=factory() 
// factory 是个匿名函数,该匿名函数并没自执行 设计参数 window，并传入window对象。不污染全局变量，也不会被别的代码污染
复制代码
```

### 第 11 行至第 111 行

// 工具代码

```bash
var emptyObject = Object.freeze({});// 冻结的对象无法再更改 [link](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
复制代码
```

// 接下来是一些封装用来判断基本类型、引用类型、类型转换的方法

- isUndef//判断未定义
- isDef// 判断已定义
- isTrue// 判断为 true
- isFalse// 判断为 false
- isPrimitive// 判断为原始类型
- isObject// 判断为 obj
- toRawType // 切割引用类型得到后面的基本类型，例如：[object RegExp] 得到的就是 RegExp
- isPlainObject// 判断纯粹的对象："纯粹的对象"，就是通过 { }、new Object()、Object.create(null) 创建的对象
- isRegExp// 判断原生引用类型
- isValidArrayIndex// 检查val是否是一个有效的数组索引，其实就是验证是否是一个非无穷大的正整数
- isPromise// 判断是否是 Promise
- toString// 类型转成 String
- toNumber// 类型转成 Number

### 第 113 行至第 354 行

- makeMap// makeMap 方法将字符串切割，放到map中，用于校验其中的某个字符串是否存在（区分大小写）于map中 e.g.

```scss
var isBuiltInTag = makeMap('slot,component', true);// 是否为内置标签
isBuiltInTag('slot'); //true
isBuiltInTag('slot1'); //undefined
var isReservedAttribute = makeMap('key,ref,slot,slot-scope,is');// 是否为保留属性
复制代码
```

- remove// 数组移除元素方法
- hasOwn// 判断对象是否含有某个属性
- cached// ※高级函数 cached函数，输入参数为函数，返回值为函数。同时使用了闭包，其会将该传入的函数的运行结果缓存，创建一个cache对象用于缓存运行fn的运行结果。[link](https://link.juejin.cn?target=http%3A%2F%2Fu-to-world.com%3A8081%2F%3Fp%3D287)

```php
function cached(fn) {
    var cache = Object.create(null);// 创建一个空对象
    return (function cachedFn(str) {// 获取缓存对象str属性的值，如果该值存在，直接返回，不存在调用一次fn，然后将结果存放到缓存对象中
        var hit = cache[str];
        return hit || (cache[str] = fn(str))
    })
}
复制代码
```

- camelize// 驼峰化一个连字符连接的字符串
- capitalize// 对一个字符串首字母大写
- hyphenateRE// 用字符号连接一个驼峰的字符串
- polyfillBind// ※高级函数 参考[link](https://link.juejin.cn?target=http%3A%2F%2Fu-to-world.com%3A8081%2F%3Fp%3D287)
- Function.prototype.bind() // [link1](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FFunction%2Fbind%23%23Polyfill)[link2](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FFunction%2Fbind)
- toArray// 将[像数组的](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Flessfish%2Funderscore-analysis%2Fissues%2F14)转为真数组
- extend// 将多个属性插入目标的对象
- toObject// 将对象数组合并为单个对象。

e.g.

```css
console.log(toObject(["bilibli"]))
//{0: "b", 1: "i", 2: "l", 3: "i", 4: "b", 5: "l", 6: "i", encodeHTML: ƒ}
复制代码
```

- no// 任何情况都返回false
- identity // 返回自身
- genStaticKeys// 从编译器模块生成包含静态键的字符串。TODO:demo
- looseEqual//※高级函数 对对象的浅相等进行判断

//有赞、头条面试题

```javascript
function looseEqual(a, b) {
    if (a === b) return true
    const isObjectA = isObject(a)
    const isObjectB = isObject(b)
    if(isObjectA && isObjectB) {
        try {
            const isArrayA = Array.isArray(a)
            const isArrayB = Array.isArray(b)
            if(isArrayA && isArrayB) {
                return a.length === b.length && a.every((e, i) => {
                    return looseEqual(e, b[i])
                })
            }else if(!isArrayA && !isArrayB) {
                const keysA = Object.keys(a)
                const keysB = Object.keys(b)
                return keysA.length === keysB.length && keysA.every(function (key) {
                    return looseEqual(a[key], b[key])
                })
            }else {
                return false
            }
        } catch(e) {
            return false
        }
    }else if(!isObjectA && !isObjectB) {
        return String(a) === String(b)
    }else {
        return false
    }
}
复制代码
```

- looseIndexOf// 返回索引，如果没找到返回-1，否则执行looseEqual()
- once// 确保函数只被调用一次，用到闭包

### 阶段小结

- cached
- polyfillBind
- looseEqual

这三个函数要重点细品！主要的点是：闭包、类型判断，函数之间的互相调用。也即是这部分工具函数的精华！

### 第 356 行 至 第 612 行

// 定义常量和配置

- SSR_ATTR// 服务端渲染
- ASSET_TYPES// 全局函数 component、directive、filter
- LIFECYCLE_HOOKS// 生命周期，无需多言
- config // 全局配置 [link](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fgreatdesert%2Fp%2F11011015.html)
- unicodeRegExp//用于解析html标记、组件名称和属性pat的unicode字母
- isReserved//  检查变量的开头是 $ 或 _
- def// 在一个对象上定义一个属性的构造函数，其中 !!enumerable 强制转换 boolean
- parsePath// 解析一个简单路径 TODO:
- userAgent// 浏览器识别
- inBrowser
- _isServer//检测 vue的服务器渲染是否存在, 而且避免webpack去填充process
- isNative //这里判断 函数是否是系统函数, 比如 Function Object ExpReg window document 等等, 这些函数应该使用c/c++实现的。这样可以区分 Symbol是系统函数, 还是用户自定义了一个Symbol
- hasSymbol//这里使用了ES6的Reflect方法, 使用这个对象的目的是, 为了保证访问的是系统的原型方法, ownKeys 保证key的输出顺序, 先数组 后字符串
- _Set// 设置一个Set

[link](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fdhsz%2Fp%2F7064913.html)

### 第 616 行至第 706 行

//设置warn,tip等全局变量 TODO:

- warn
- tip
- generateComponentTrace// 生成组件跟踪路径（组件数规则）
- formatComponentName// 格式化组件名

### 第 710 行至第 763 行

**Vue核心：数据监听最重要之一的 Dep**

```ini
// Dep是订阅者Watcher对应的数据依赖
var Dep = function Dep () {
  //每个Dep都有唯一的ID
  this.id = uid++;
  //subs用于存放依赖
  this.subs = [];
};

//向subs数组添加依赖
Dep.prototype.addSub = function addSub (sub) {
  this.subs.push(sub);
};
//移除依赖
Dep.prototype.removeSub = function removeSub (sub) {
  remove(this.subs, sub);
};
//设置某个Watcher的依赖
//这里添加了Dep.target是否存在的判断，目的是判断是不是Watcher的构造函数调用
//也就是说判断他是Watcher的this.get调用的，而不是普通调用
Dep.prototype.depend = function depend () {
  if (Dep.target) {
    Dep.target.addDep(this);
  }
};

Dep.prototype.notify = function notify () {
  var subs = this.subs.slice();
  //通知所有绑定 Watcher。调用watcher的update()
  for (var i = 0, l = subs.length; i &lt; l; i++) {
    subs[i].update();
  }
};
复制代码
```

强烈推荐阅读：[link](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fdatiangou%2Fp%2F10144883.html)

Dep 相当于把 Observe 监听到的信号做一个收集（collect dependencies），然后通过dep.notify()再通知到对应 Watcher ，从而进行视图更新。

### 第 767 行至第 900 行

**Vue核心：视图更新最重要的 VNode（ Virtual DOM）**

- VNode
- createEmptyVNode
- createTextVNode
- cloneVNode

把你的 template 模板 描述成 VNode，然后一系列操作之后通过 VNode 形成真实DOM进行挂载

更新的时候对比旧的VNode和新的VNode，只更新有变化的那一部分，提高视图更新速度。

e.g.

```arduino
<div class="parent" style="height:0" href="2222">
    111111
</div>

//转成Vnode
{    

    tag: 'div',    

    data: {        

        attrs:{href:"2222"}

        staticClass: "parent",        

        staticStyle: {            

            height: "0"

        }
    },    

    children: [{        

        tag: undefined,        

        text: "111111"

    }]
}
复制代码
```

强烈推荐阅读：[link](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1479295)

- methodsToPatch

将数组的基本操作方法拓展，实现响应式，视图更新。

因为：对于对象的修改是可以直接触发响应式的，但是对数组直接赋值，是无法触发的，但是用到这里经过改造的方法。我们可以明显的看到 ob.dep.notify() 这一核心。

### 阶段小结

这一 part 最重要的，毋庸置疑是：Dep 和 VNode，需重点突破！！！

### 第 904 行至第 1073 行

**Vue核心：数据监听最重要之一的 Observer**

- 核心的核心！Observer（发布者） => Dep（订阅器） => Watcher（订阅者）

类比一个生活场景：报社将各种时下热点的新闻收集，然后制成各类报刊，发送到每家门口的邮箱里，订阅报刊人们看到了新闻，对新闻作出评论。

在这个场景里，报社==发布者，新闻==数据，邮箱==订阅器，订阅报刊的人==订阅者，对新闻评论==视图更新

- Observer//Observer的调用过程：initState()-->observe(data)-->new Observer()

```ini
var Observer = function Observer (value) {
  this.value = value;
  this.dep = new Dep();
  this.vmCount = 0;
  def(value, '__ob__', this);
  if (Array.isArray(value)) {
    if (hasProto) {
      protoAugment(value, arrayMethods);
    } else {
      copyAugment(value, arrayMethods, arrayKeys);
    }
    this.observeArray(value);
  } else {
    this.walk(value);
  }
};
复制代码
```

- ※※ defineReactive 函数，定义一个响应式对象，给对象动态添加 getter 和 setter ，用于依赖收集和派发更新。

```vbnet
function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()// 1. 为属性创建一个发布者

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get // 依赖收集
  const setter = property && property.set // 派发更新
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val)// 2. 获取属性值的__ob__属性
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()// 3. 添加 Dep
        if (childOb) {
          childOb.dep.depend()//4. 也为属性值添加同样的 Dep 
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
复制代码
```

第 4 步非常重要。为对象的属性添加 dep.depend(),达到监听对象（引用的值）属性的目的

### 重点备注

Vue对数组的处理跟对象还是有挺大的不同，length是数组的一个很重要的属性，无论数组增加元素或者删除元素（通过splice，push等方法操作）length的值必定会更新，为什么不直接操作监听length呢？而需要拦截splice，push等方法进行数组的状态更新？

原因是：在数组length属性上用defineProperty拦截的时候，会报错。

```javascript
Uncaught TypeError: Cannot redefine property: length
复制代码
```

再用Object.getOwnPropertyDescriptor(arr, 'length')查看一下：//（Object.getOwnPropertyDescriptor用于返回defineProperty.descriptor）

{ configurable: false enumerable: false value: 0 writable: true } configurable为false，且MDN上也说重定义数组的length属性在不同浏览器上表现也是不一致的，所以还是老老实实拦截splice，push等方法，或者使用ES6的Proxy。

### 第 1075 行至第 1153 行

- set //在对象上设置一个属性。如果是新的属性就会触发更改通知（旧属性也会触发更新通知，因为第一个添加的时候已经监听了，之后自动触发，不再手动触发）
- del //删除一个属性，如果必要触发通知
- dependArray // 收集数组的依赖 [link](https://link.juejin.cn?target=http%3A%2F%2Fqjzd.net%3A3000%2Ftopic%2F57cd26dad703dbd15b10c707)

### 第 1157 行至第 1568 行

// 配置选项合并策略

```ini
ar strats = config.optionMergeStrategies;
复制代码
```

- mergeData
- strats.data
- mergeDataOrFn
- mergeHook
- mergeAssets
- strats.watch
- strats.computed
- defaultStrat
- checkComponents
- validateComponentName
- normalizeProps
- normalizeInject
- normalizeDirectives
- assertObjectType
- mergeOptions

这一部分代码写的就是父子组件配置项的合并策略，包括：默认的合并策略、钩子函数的合并策略、filters/props、data合并策略，且包括标准的组件名、props写法有一个统一化规范要求。

一图以蔽之



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/173237ee5b806a7f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



强烈推荐阅读：[link](https://link.juejin.cn?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000014738314%23item-1)

### 阶段小结

这一部分最重要的就是 Observer（观察者） ，这也是 Vue 核心中的核心！其次是 mergeOptions（组件配置项的合并策略），但是通常在用的过程中，就已经了解到了大部分的策略规则。

### 第 1570 行至第 1754 行

- resolveAsset// resolveAsset 全局注册组件用到

e.g.

我们的调用 resolveAsset(context.![options, 'components', tag)，即拿 vm.](https://juejin.cn/equation?tex=options%2C%20'components'%2C%20tag)%EF%BC%8C%E5%8D%B3%E6%8B%BF%20vm.)options.components[tag]，这样我们就可以在 resolveAsset 的时候拿到这个组件的构造函数，并作为 createComponent 的钩子的参数。

- validateProp// prop的格式校验

校验prop：

1. prop为Boolean类型时做特殊处理
2. prop的值为空时，获取默认值，并创建观察者对象
3. prop验证

- getPropDefaultValue// 获取默认 prop 值

获取 prop 的默认值 && 创建观察者对象

1. @param {*} vm vm 实例
2. @param {*} prop 定义选项
3. @param {*} vmkey prop 的 key

// 在非生产环境下（除去 Weex 的某种情况），将对prop进行验证，包括验证required、type和自定义验证函数。

- assertProp //验证 prop Assert whether a prop is valid.

```typescript
case 1: 验证 required 属性
   case 1.1: prop 定义时是 required，但是调用组件时没有传递该值（警告）
   case 1.2: prop 定义时是非 required 的，且 value === null || value === undefined（符合要求，返回）
case 2: 验证 type 属性-- value 的类型必须是 type 数组里的其中之一
case 3: 验证自定义验证函数
复制代码
```

- assertType

```markdown
`assertType`函数，验证`prop`的值符合指定的`type`类型，分为三类：
  - 第一类：通过`typeof`判断的类型，如`String`、`Number`、`Boolean`、`Function`、`Symbol`
  - 第二类：通过`Object.prototype.toString`判断`Object`/`Array`
  - 第三类：通过`instanceof`判断自定义的引用类型
复制代码
```

### 第 1756 行至第 1823 行

// 辅助函数：检测内置类型

- getType
- isSameType
- getTypeIndex
- getInvalidTypeMessage
- styleValue
- isExplicable
- isBoolean

### 第 1827 行至第 1901 行

// 辅助函数：处理错误、错误打印

- handleError
- invokeWithErrorHandling
- globalHandleError
- logError

### 第 1905 行至第 2007 行

- flushCallbacks// flushCallbacks 挨个同步执行callbacks中回调
- MutationObserver
- nextTick// 把传入的 cb 回调函数用 try-catch 包裹后放在一个匿名函数中推入callbacks数组中，这么做是因为防止单个 cb 如果执行错误不至于让整个JS线程挂掉，每个 cb 都包裹是防止这些回调函数如果执行错误不会相互影响，比如前一个抛错了后一个仍然可以执行。

**精髓中的精髓 —— nextTick**

这里有一段很重要的注释

```less
// Here we have async deferring wrappers using microtasks.
// In 2.5 we used (macro) tasks (in combination with microtasks).
// However, it has subtle problems when state is changed right before repaint
// (e.g. #6813, out-in transitions).
// Also, using (macro) tasks in event handler would cause some weird behaviors
// that cannot be circumvented (e.g. #7109, #7153, #7546, #7834, #8109).
// So we now use microtasks everywhere, again.
// A major drawback of this tradeoff is that there are some scenarios
// where microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690, which have workarounds)
// or even between bubbling of the same event (#6566).

在vue2.5之前的版本中，nextTick基本上基于 micro task 来实现的，但是在某些情况下 micro task 具有太高的优先级，并且可能在连续顺序事件之间（例如＃4521，＃6690）或者甚至在同一事件的事件冒泡过程中之间触发（＃6566）。但是如果全部都改成 macro task，对一些有重绘和动画的场景也会有性能影响，如 issue #6813。vue2.5之后版本提供的解决办法是默认使用 micro task，但在需要时（例如在v-on附加的事件处理程序中）强制使用 macro task。
复制代码
```

什么意思呢？分析下面这段代码。

```kotlin
<span id='name' ref='name'>{{ name }}</span>
<button @click='change'>change name</button>

methods: {
      change() {
          this.$nextTick(() => console.log('setter前：' + this.$refs.name.innerHTML))
          this.name = ' vue3 '
          console.log('同步方式：' + this.$refs.name.innerHTML)
          setTimeout(() => this.console("setTimeout方式：" + this.$refs.name.innerHTML))
          this.$nextTick(() => console.log('setter后：' + this.$refs.name.innerHTML))
          this.$nextTick().then(() => console.log('Promise方式：' + this.$refs.name.innerHTML))
      }
  }
//同步方式：vue2
//setter前：vue2
//setter后： vue3 
//Promise方式： vue3 
//setTimeout方式： vue3 
复制代码
```

1. 同步方式： 当把data中的name修改之后，此时会触发name的 setter 中的 dep.notify 通知依赖本data的render watcher去 update，update 会把 flushSchedulerQueue 函数传递给 nextTick，render watcher在 flushSchedulerQueue 函数运行时 watcher.run 再走 diff -> patch 那一套重渲染 re-render 视图，这个过程中会重新依赖收集，这个过程是异步的；所以当我们直接修改了name之后打印，这时异步的改动还没有被 patch 到视图上，所以获取视图上的DOM元素还是原来的内容。
2. setter前： setter前为什么还打印原来的是原来内容呢，是因为 nextTick 在被调用的时候把回调挨个push进callbacks数组，之后执行的时候也是 for 循环出来挨个执行，所以是类似于队列这样一个概念，先入先出；在修改name之后，触发把render watcher填入 schedulerQueue 队列并把他的执行函数 flushSchedulerQueue 传递给 nextTick ，此时callbacks队列中已经有了 setter前函数 了，因为这个 cb 是在 setter前函数 之后被push进callbacks队列的，那么先入先出的执行callbacks中回调的时候先执行 setter前函数，这时并未执行render watcher的 watcher.run，所以打印DOM元素仍然是原来的内容。
3. setter后： setter后这时已经执行完 flushSchedulerQueue，这时render watcher已经把改动 patch 到视图上，所以此时获取DOM是改过之后的内容。
4. Promise方式： 相当于 Promise.then 的方式执行这个函数，此时DOM已经更改。
5. setTimeout方式： 最后执行macro task的任务，此时DOM已经更改。

备注：前文提过，在依赖收集原理的响应式化方法 defineReactive 中的 setter 访问器中有派发更新 dep.notify() 方法，这个方法会挨个通知在 dep 的 subs 中收集的订阅自己变动的 watchers 执行 update。

强烈推荐阅读：[link](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F55423103)

### 0 行 至 2000 行小结

0 至 2000 行主要的内容是：

1. 工具代码
2. 数据监听：Obeserve,Dep
3. Vnode
4. nextTick

### 第 2011 行至第 2232 行

- perf// performance
- initProxy// 代理对象是es6的新特性，它主要用来自定义对象一些基本操作（如查找，赋值，枚举等）。[link](https://juejin.cn/post/6844903615589515272)

//proxy是一个强大的特性，为我们提供了很多"元编程"能力。

```ini
const handler = {
    get: function(obj, prop) {
        return prop in obj ? obj[prop] : 37;
    }
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b);      // 1, undefined
console.log('c' in p, p.c); // false, 37
复制代码
```

[link](https://juejin.cn/post/6844903545599164423)

- traverse// 遍历：_traverse 深度遍历，用于

traverse 对一个对象做深层递归遍历，因为遍历过程中就是对一个子对象的访问，会触发它们的 getter 过程，这样就可以收集到依赖，也就是订阅它们变化的 watcher，且遍历过程中会把子响应式对象通过它们的 dep id 记录到 seenObjects，避免以后重复访问。

- normalizeEvent// normalizeEvents是针对v-model的处理,例如在IE下不支持change事件，只能用input事件代替。
- createFnInvoker// 在初始构建实例时，旧节点是不存在的,此时会调用createFnInvoker函数对事件回调函数做一层封装，由于单个事件的回调可以有多个，因此createFnInvoker的作用是对单个，多个回调事件统一封装处理，返回一个当事件触发时真正执行的匿名函数。
- updateListeners// updateListeners的逻辑也很简单，它会遍历on事件对新节点事件绑定注册事件，对旧节点移除事件监听，它即要处理原生DOM事件的添加和移除，也要处理自定义事件的添加和移除，

### 阶段小结

[Vue 的事件机制](https://juejin.cn/post/6844903919290679304#heading-7)

### 第 2236 行至第 2422 行

- mergeVNodeHook// 重点 合并 VNode

// 把 hook 函数合并到 def.data.hook[hookey] 中，生成新的 invoker，createFnInvoker 方法

// vnode 原本定义了 init、prepatch、insert、destroy 四个钩子函数，而 mergeVNodeHook 函数就是把一些新的钩子函数合并进来，例如在 transition 过程中合并的 insert 钩子函数，就会合并到组件 vnode 的 insert 钩子函数中，这样当组件插入后，就会执行我们定义的 enterHook 了。

- extractPropsFromVNodeData// 抽取相应的从父组件上的prop
- checkProp// 校验 Prop

```less
    // The template compiler attempts to minimize the need for normalization by
    // statically analyzing the template at compile time.
    // 模板编译器尝试用最小的需求去规范：在编译时，静态分析模板

    // For plain HTML markup, normalization can be completely skipped because the
    // generated render function is guaranteed to return Array<VNode>. There are
    // two cases where extra normalization is needed:
    // 对于纯 HTML 标签，可跳过标准化，因为生成渲染函数一定会会返回 Vnode Array.有两种情况，需要额外去规范

    // 1. When the children contains components - because a functional component
    // may return an Array instead of a single root. In this case, just a simple
    // normalization is needed - if any child is an Array, we flatten the whole
    // thing with Array.prototype.concat. It is guaranteed to be only 1-level deep
    // because functional components already normalize their own children.
    // 当子级包含组件时-因为功能组件可能会返回Array而不是单个根。在这种情况下，需要规范化-如果任何子级是Array，我们将整个具有Array.prototype.concat的东西。保证只有1级深度，因为功能组件已经规范了自己的子代。

    // 2. When the children contains constructs that always generated nested Arrays,
    // e.g. <template>, <slot>, v-for, or when the children is provided by user
    // with hand-written render functions / JSX. In such cases a full normalization
    // is needed to cater to all possible types of children values.
    // 当子级包含始终生成嵌套数组的构造时，例如<template>，<slot>，v-for或用户提供子代时,具有手写的渲染功能/ JSX。在这种情况下，完全归一化,才能满足所有可能类型的子代值。
复制代码
```

Q:这一段话说的是什么意思呢？

A：归一化操作其实就是将多维的数组，合并转换成一个一维的数组。在 Vue 中归一化分为三个级别，

1. 不需要进行归一化
2. 只需要简单的归一化处理，将数组打平一层
3. 完全归一化，将一个 N 层的 children 完全打平为一维数组

利用递归来处理的，同时处理了一些边界情况。

### 第 2426 行至第 2490 行

- initProvide
- initInjections
- resolveInject

### 第 2497 行至第 2958 行

- resolveSlots// Runtime helper for resolving raw children VNodes into a slot object.
- isWhitespace
- normalizeScopedSlots
- normalizeScopedSlot
- proxyNormalSlot
- renderList// Runtime helper for rendering v-for lists.
- renderSlot// Runtime helper for rendering ``
- resolveFilter// Runtime helper for resolving filters
- checkKeyCodes// Runtime helper for checking keyCodes from config.
- bindObjectProps//  Runtime helper for merging v-bind="object" into a VNode's data.
- renderStatic// Runtime helper for rendering static trees.
- markOnce// Runtime helper for v-once.

这一部分讲的是辅助程序 —— Vue 的各类渲染方法，从字面意思中可以知道一些方法的用途，这些方法用在Vue生成的渲染函数中。

- installRenderHelpers// installRenderHelpers 用于执行以上。

### 第 2962 行至第 3515 行

- FunctionalRenderContext// 创建一个包含渲染要素的函数
- createFunctionalComponent

函数式组件的实现

```kotlin
  Ctor,                                       //Ctro:组件的构造对象(Vue.extend()里的那个Sub函数)
  propsData,                                  //propsData:父组件传递过来的数据(还未验证)
  data,                                       //data:组件的数据
  contextVm,                                  //contextVm:Vue实例 
  children                                    //children:引用该组件时定义的子节点
复制代码
```

// createFunctionalComponent 最后会执行我们的 render 函数

特注：Vue 组件是 Vue 的核心之一

组件分为：异步组件和函数式组件

这里就是**函数式组件相关**

> Vue提供了一种可以让组件变为无状态、无实例的函数化组件。从原理上说，一般子组件都会经过实例化的过程，而单纯的函数组件并没有这个过程，它可以简单理解为一个中间层，只处理数据，不创建实例，也是由于这个行为，它的渲染开销会低很多。实际的应用场景是，当我们需要在多个组件中选择一个来代为渲染，或者在将children,props,data等数据传递给子组件前进行数据处理时，我们都可以用函数式组件来完成，它本质上也是对组件的一个外部包装。

函数式组件会在组件的对象定义中，将functional属性设置为true，这个属性是区别普通组件和函数式组件的关键。同样的在遇到子组件占位符时，会进入createComponent进行子组件Vnode的创建。**由于functional属性的存在，代码会进入函数式组件的分支中，并返回createFunctionalComponent调用的结果。**注意，执行完createFunctionalComponent后，后续创建子Vnode的逻辑不会执行，这也是之后在创建真实节点过程中不会有子Vnode去实例化子组件的原因。(无实例)

[官方说明](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcomponents-dynamic-async.html)

- cloneAndMarkFunctionalResult
- mergeProps
- componentVNodeHooks
- createComponent // createComponent 方法创建一个组件的 VNode。这 createComponent 是创建子组件的关键

// 创建组件的 VNode 时，若组件是函数式组件，则其 VNode 的创建过程将与普通组件有所区别。

- createComponentInstanceForVnode // [link](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FHcySunYang%2Fvue-design%2Fissues%2F199)

推荐阅读：[link](https://juejin.cn/post/6844903811371237384#heading-7)

- installComponentHooks // installComponentHooks就是把 componentVNodeHooks的钩子函数合并到data.hook中，，在合并过程中，如果某个时机的钩子已经存在data.hook中，那么通过执行mergeHook函数做合并勾子。
- mergeHook$1
- transformModel
- createElement// 创建元素
- _createElement
- applyNS
- registerDeepBindings
- initRender // 初识渲染

[link](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F79538534)

### 阶段小结

这一部分主要是围绕 Vue 的组件的创建。Vue 将页面划分成各类的组件，组件思想是 Vue 的精髓之一。

### 第 3517 行至第 3894 行

- renderMixin // 引入视图渲染混合函数
- ensureCtor
- createAsyncPlaceholder
- resolveAsyncComponent
- isAsyncPlaceholder
- getFirstComponentChild
- initEvents// 初始化事件
- add
- remove$1
- createOnceHandler
- updateComponentListeners
- eventsMixin // 挂载事件响应相关方法

### 第 3898 行至第 4227 行

- setActiveInstance
- initLifecycle
- lifecycleMixin// 挂载生命周期相关方法
- mountComponent
- updateChildComponent
- isInInactiveTree
- activateChildComponent
- deactivateChildComponent
- callHook

> 几乎所有JS框架或插件的编写都有一个类似的模式，即向全局输出一个类或者说构造函数，通过创建实例来使用这个类的公开方法，或者使用类的静态全局方法辅助实现功能。相信精通Jquery或编写过Jquery插件的开发者会对这个模式非常熟悉。Vue.js也如出一辙，只是一开始接触这个框架的时候对它所能实现的功能的感叹盖过了它也不过是一个内容较为丰富和精致的大型类的本质。

[link](https://juejin.cn/post/6844903775291834381)

### 阶段小结

这里要对 js 的继承有一个深刻的理解。 [link](https://juejin.cn/post/6844903472433725447)

1. 类继承

```javascript
function Animal(){
    this.live=true;
}
function Dog(name){
    this.name=name
}
Dog.prototype=new Animal()
var dog1=new Dog("wangcai")
console.log(dog1)// Dog {name: "wangcai"}
console.log(dog1.live)// true
复制代码
```

1. 构造继承

```javascript
function Animal(name,color){
    this.name=name;
    this.color=color;}
function Dog(){
    Animal.apply(this,arguments)
}
var dog1=new Dog("wangcai","balck")
console.log(dog1)// Dog {name: "wangcai", color: "balck"}
复制代码
```

1. 组合继承（类继承 + 构造继承）

```javascript
function Animal(name,color){
    this.name=name;
    this.color=color;
    this.live=true;
}
function Dog(){
    Animal.apply(this, arguments);   
}
Dog.prototype=new Animal()
var dog1=new Dog("wangcai","black")
console.log(dog1)// Dog {name: "wangcai", color: "black", live: true}
复制代码
```

1. 寄生组合式继承
2. extend继承

Vue 同 Jquery 一样，本质也是一个大型的类库。

// 定义Vue构造函数，形参options

```arduino
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // ...  
  this._init(options)
}
复制代码
```

// 功能函数

```javascript
// 引入初始化混合函数
import { initMixin } from './init'
// 引入状态混合函数
import { stateMixin } from './state'
// 引入视图渲染混合函数
import { renderMixin } from './render'
// 引入事件混合函数
import { eventsMixin } from './events'
// 引入生命周期混合函数
import { lifecycleMixin } from './lifecycle'
// 引入warn控制台错误提示函数
import { warn } from '../util/index'
...

// 挂载初始化方法
initMixin(Vue)
// 挂载状态处理相关方法
stateMixin(Vue)
// 挂载事件响应相关方法
eventsMixin(Vue)
// 挂载生命周期相关方法
lifecycleMixin(Vue)
// 挂载视图渲染方法
renderMixin(Vue)
复制代码
```

### 第 4231 行至第 4406 行

- resetSchedulerState // 重置状态
- flushSchedulerQueue// 据变化最终会把flushSchedulerQueue传入到nextTick中执行flushSchedulerQueue函数会遍历执行watcher.run()方法，watcher.run()方法最终会完成视图更新



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/17323802c9a3b2d0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



vue中dom的更像并不是实时的，当数据改变后，vue会把渲染watcher添加到异步队列，异步执行，同步代码执行完成后再统一修改dom。

- callUpdatedHooks
- queueActivatedComponent
- callActivatedHooks
- queueWatcher

[link](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1356678)

### 第 4412 行至第 4614 行

- Watcher// !important 重中之重的重点

这一 part 在 Watcher 的原型链上定义了get、addDep、cleanupDeps、update、run、evaluate、depend、teardown 方法，即 Watcher 的具体实现的一些方法，比如新增依赖、清除、更新试图等。

每个Vue组件都有一个对应的watcher，这个watcher将会在组件render的时候收集组件所依赖的数据，并在依赖有更新的时候，触发组件重新渲染。

### 第 4618 行至第 5071 行

```scss
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }
    // 如果是Vue的实例，则不需要被observe
    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    // 第一步： options参数的处理
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      // mergeOptions接下来我们会详细讲哦~
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    // 第二步： renderProxy
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    // 第三步： vm的生命周期相关变量初始化
    initLifecycle(vm)
    
    // 第四步： vm的事件监听初始化
    initEvents(vm)
    // 第五步： vm的编译render初始化
    initRender(vm)
    // 第六步： vm的beforeCreate生命钩子的回调
    callHook(vm, 'beforeCreate')
    // 第七步： vm在data/props初始化之前要进行绑定
    initInjections(vm) // resolve injections before data/props
    
    // 第八步： vm的sate状态初始化
    initState(vm)
    // 第九步： vm在data/props之后要进行提供
    initProvide(vm) // resolve provide after data/props
    // 第十步： vm的created生命钩子的回调
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }
    // 第十一步：render & mount
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
复制代码
```

主要是为我们的Vue原型上定义一个方法_init。然后当我们执行new Vue(options) 的时候，会调用这个方法。而这个_init方法的实现，便是我们需要关注的地方。 前面定义vm实例都挺好理解的，主要我们来看一下mergeOptions这个方法，其实Vue在实例化的过程中，会在代码运行后增加很多新的东西进去。我们把我们传入的这个对象叫options，实例中我们可以通过vm.$options访问到。

[link](https://juejin.cn/post/6844903687022706702)

### 0 至 5000 行 总结



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/17323808ba0442e2~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



从 0 至 5000 行我们可以清晰看到 Vue 模板编译的轮廓了。

- 笔者将这一部分出现的关键词进行按顺序罗列：

1. function (global, factory)
2. 工具函数
3. Dep
4. Observe
5. VNode
6. nextTick
7. 事件机制
8. Render
9. components
10. Watcher

我们可以总结：Vue 的核心就是 VDOM ！对 DOM 对象的操作调整为操作 VNode 对象，采用 diff 算法比较差异，一次 patch。

render 的流程是:

1. Vue使用HTML的Parser将HTML模板解析为AST
2. function render(){}
3. Virtual DOM
4. watcher将会在组件render的时候收集组件所依赖的数据，并在依赖有更新的时候，触发组件重新渲染

推荐阅读：[link](https://link.juejin.cn?target=https%3A%2F%2Fgershonv.github.io%2F2018%2F07%2F04%2Fvue-render%2F)

### 第 5073 行至第 5446 行

```scss
// 定义 Vue 构造函数
function Vue (options) {
      if (!(this instanceof Vue)
      ) {
        warn('Vue is a constructor and should be called with the `new` keyword');
      }
      this._init(options);
    }

// 将 Vue 作为参数传递给导入的五个方法
initMixin(Vue);// 初始化 Mixin
stateMixin(Vue);// 状态 Mixin
eventsMixin(Vue);// 事件 Mixin
lifecycleMixin(Vue);// 生命周期 Mixin
renderMixin(Vue);// 渲染 Mixin
复制代码
```

这一部分就是初始化函数的调用。

```dart
// 
Object.defineProperty(Vue.prototype, '$isServer', {
      get: isServerRendering
    });
复制代码
```

为什么这么写？

Object.defineProperty能保护引入的库不被重新赋值，如果你尝试重写，程序会抛出“TypeError: Cannot assign to read only property”的错误。

[link-【译】Vue框架引入JS库的正确姿势](https://link.juejin.cn?target=https%3A%2F%2Fwufenfen.github.io%2F2017%2F04%2F24%2F%E3%80%90%E8%AF%91%E3%80%91Vue%E6%A1%86%E6%9E%B6%E5%BC%95%E5%85%A5JS%E5%BA%93%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF%2F)

```ini
// 版本
Vue.version = '2.6.11';
复制代码
```

### 阶段小结

这一部分是 Vue index.js 的内容,包括 Vue 的整个挂在过程

1. 先进入 initMixin(Vue),在prototype上挂载

```javascript
Vue.prototype._init = function (options) {} 
复制代码
```

1. 进入 stateMixin(Vue),在prototype上挂载 Vue.prototype.$data

```ini
Vue.prototype.$props 
Vue.prototype.$set = set 
Vue.prototype.$delete = del 
Vue.prototype.$watch = function(){} 
复制代码
```

1. 进入eventsMixin(Vue),在prototype上挂载

```bash
Vue.prototype.$on 
Vue.prototype.$once 
Vue.prototype.$off 
Vue.prototype.$emit
复制代码
```

1. 进入lifecycleMixin(Vue),在prototype上挂载

```javascript
Vue.prototype._update 
Vue.prototype.$forceUpdate 
Vue.prototype.$destroy
复制代码
```

1. 最后进入renderMixin(Vue),在prototype上挂载 Vue.prototype.$nextTick

```ini
Vue.prototype._render 
Vue.prototype._o = markOnce 
Vue.prototype._n = toNumber 
Vue.prototype._s = toString 
Vue.prototype._l = renderList 
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual 
Vue.prototype._i = looseIndexOf 
Vue.prototype._m = renderStatic 
Vue.prototype._f = resolveFilter 
Vue.prototype._k = checkKeyCodes 
Vue.prototype._b = bindObjectProps 
Vue.prototype._v = createTextVNode 
Vue.prototype._e = createEmptyVNode 
Vue.prototype._u = resolveScopedSlots 
Vue.prototype._g = bindObjectListeners
复制代码
```

> mergeOptions使用策略模式合并传入的options和Vue.options合并后的代码结构, 可以看到通过合并策略components,directives,filters继承了全局的, 这就是为什么全局注册的可以在任何地方使用,因为每个实例都继承了全局的, 所以都能找到。

推荐阅读：

[link](https://juejin.cn/post/6844903601953849352#heading-16)

[link](https://juejin.cn/post/6844903687022706702#heading-0)

new 一个 Vue 对象发生了什么：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/1732380da01b4abe~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



### 第 5452 行至第 5655 行

```arduino
// these are reserved for web because they are directly compiled away
// during template compilation

// 这些是为web保留的，因为它们是直接编译掉的
// 在模板编译期间
复制代码
```

- isBooleanAttr
- genClassForVnode// class 转码获取vonde 中的staticClass 静态class  和class动态class转义成真实dom需要的class格式。然后返回class字符串
- mergeClassData// mergeClassData
- renderClass// 渲染calss 这里获取到已经转码的calss
- stringifyClass// 转码 class，把数组格式，对象格式的calss 全部转化成 字符串格式
- stringifyArray// 数组字符串变成字符串，然后用空格 隔开 拼接 起来变成字符串
- stringifyObject// 对象字符串变成字符串，然后用空格 隔开 拼接 起来变成字符串
- namespaceMap
- isHTMLTag
- isSVG// 判断svg 标签
- isUnknownElement// 检查dom 节点的tag标签 类型 是否是VPre 标签 或者是判断是否是浏览器自带原有的标签
- isTextInputType //  //匹配'text,number,password,search,email,tel,url'

这一 part 没有特别要说的，主要是对 class 的转码、合并和其他二次封装的工具函数。实际上我们在 Vue 源码很多地方看到了这样的封装，在平常的开发中，我们也得要求自己封装基本的函数。如果能形成自己习惯用的函数的库，会方便很多，且对自己能力也是一个提升。

### 第 5659 行至第 5792 行

- createElement // 创建元素，实例化 VNode
- createElementNS
- createTextNode
- createComment
- insertBefore
- removeChild
- appendChild
- parentNode
- nextSibling
- tagName
- setTextContent
- setStyleScope
- nodeOps

```swift
// nodeOps:
    createElement: createElement$1, //创建一个真实的dom
    createElementNS: createElementNS, //创建一个真实的dom svg方式
    createTextNode: createTextNode, // 创建文本节点
    createComment: createComment,  // 创建一个注释节点
    insertBefore: insertBefore,  //插入节点 在xxx  dom 前面插入一个节点
    removeChild: removeChild,   //删除子节点
    appendChild: appendChild,  //添加子节点 尾部
    parentNode: parentNode,  //获取父亲子节点dom
    nextSibling: nextSibling,     //获取下一个兄弟节点
    tagName: tagName,   //获取dom标签名称
    setTextContent: setTextContent, //  //设置dom 文本
    setStyleScope: setStyleScope  //设置组建样式的作用域
复制代码
```

- ref
- registerRef // 注册ref或者删除ref。比如标签上面设置了ref='abc' 那么该函数就是为this.$refs.abc 注册ref 把真实的dom存进去

### 阶段小结

这里的重点想必就是 “ref” 了

在绝大多数情况下，我们最好不要触达另一个组件实例内部或手动操作 DOM 元素。不过也确实在一些情况下做这些事情是合适的。ref 为我们提供了解决途径。

ref属性不是一个标准的HTML属性，只是Vue中的一个属性。

### 第 5794 行至第 6006 行

Virtual DOM !

没错，这里就是 虚拟 dom 生成的源码相关。

- sameVnode
- sameInputType
- createKeyToOldIdx
- createPatchFunction // !important:patch 把 vonde 渲染成真实的 dom
- emptyNodeAt
- createRmCb
- removeNode
- isUnknownElement?1
- createElm // 创造 dom 节点
- createComponent // 创建组件，并且判断它是否实例化过
- initComponent

> createElement方法接收一个tag参数，在内部会去判断tag标签的类型，从而去决定是创建一个普通的VNode还是一个组件类VNode；

createComponent 的实现，在渲染一个组件的时候的 3 个关键逻辑：

1. 构造子类构造函数，
2. 安装组件钩子函数
3. 实例化 vnode。createComponent 后返回的是组件 vnode，它也一样走到 vm._update 方法

我们传入的 vnode 是组件渲染的 vnode，也就是我们之前说的 vm._vnode，如果组件的根节点是个普通元素，那么 vm._vnode 也是普通的 vnode，这里 createComponent(vnode, insertedVnodeQueue, parentElm, refElm) 的返回值是 false。接下来的过程就系列一的步骤一样了，先创建一个父节点占位符，然后再遍历所有子 VNode 递归调用 createElm，在遍历的过程中，如果遇到子 VNode 是一个组件的 VNode，则重复过程，这样通过一个递归的方式就可以完整地构建了整个组件树。

> initComponent 初始化组建，如果没有tag标签则去更新真实dom的属性，如果有tag标签，则注册或者删除ref 然后为insertedVnodeQueue.push(vnode);

[参考link](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fhao123456%2Fp%2F10616356.html)

### 第 6008 行至第 6252 行

- reactivateComponent
- insert
- createChildren
- isPatchable
- invokeCreateHooks
- setScope
- addVnodes // 添加 Vnodes
- invokeDestroyHook
- removeVnodes // 移除 Vnodes
- removeAndInvokeRemoveHook
- updateChildren // 在patchVnode中提到，如果新老节点都有子节点，但是不相同的时候就会调用 updateChildren，这个函数通过diff算法尽可能的复用先前的DOM节点。

// diff 算法就在这里辣！[详解link](https://juejin.cn/post/6844903607913938951)

```scss
function updateChildren(parentElm, oldCh, newCh, insertedVnodeQueue) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, elmToMove, refElm 
    
    while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (isUndef(oldStartVnode)) {
            oldStartVnode = oldCh[++oldStartIdx]
        } else if (isUndef(oldEndVnode)) {
            oldEndVnode = oldCh[--oldEndIdx]
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
            oldStartVnode = oldCh[++oldStartIdx]
            newStartVnode = newCh[++newStartIdx]
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
            oldEndVnode = oldCh[--oldEndIdx]
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
            canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
            oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
            canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
            oldEndVnode = oldCh[--oldEndIdx]
            newStartVnode = newCh[++newStartIdx]
        } else {
            if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
            idxInOld = isDef(newStartVnode.key) ? oldKeyToIdx[newStartVnode.key] : null
            if (isUndef(idxInOld)) {
                createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
                newStartVnode = newCh[++newStartIdx]
            } else {
                elmToMove = oldCh[idxInOld]
                if (sameVnode(elmToMove, newStartVnode)) {
                    patchVnode(elmToMove, newStartVnode, insertedVnodeQueue)
                    oldCh[idxInOld] = undefined
                    canMove && nodeOps.insertBefore(parentElm, newStartVnode.elm, oldStartVnode.elm)
                    newStartVnode = newCh[++newStartIdx]
                } else {
                    createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
                    newStartVnode = newCh[++newStartIdx]
                }
            }
        }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
}
复制代码
```

- checkDuplicateKeys
- findIdxInOld

reactivateComponent 承接上文 createComponent

### 第 6259 行至第 6561 行

- patchVnode // 如果符合sameVnode，就不会渲染vnode重新创建DOM节点，而是在原有的DOM节点上进行修补，尽可能复用原有的DOM节点。
- invokeInsertHook
- isRenderedModule
- hydrate
- assertNodeMatch
- patch // !important: patch的本质是将新旧vnode进行比较，创建、删除或者更新DOM节点/组件实例

### 阶段小结

Vue 的核心思想：组件化。

这一部分是关于构建组件树，形成虚拟 dom ，以及非常重要的 patch 方法。

再来亿遍：

1. 原因：当修改某条数据的时候，这时候js会将整个DOM Tree进行替换，这种操作是相当消耗性能的。所以在Vue中引入了Vnode的概念：Vnode是对真实DOM节点的模拟，可以对Vnode Tree进行增加节点、删除节点和修改节点操作。这些过程都只需要操作VNode Tree，不需要操作真实的DOM，大大的提升了性能。修改之后使用diff算法计算出修改的最小单位，在将这些小单位的视图进行更新。
2. 原理：data中定义了一个变量a，并且模板中也使用了它，那么这里生成的Watcher就会加入到a的订阅者列表中。当a发生改变时，对应的订阅者收到变动信息，这时候就会触发Watcher的update方法，实际update最后调用的就是在这里声明的updateComponent。 当数据发生改变时会触发回调函数updateComponent，updateComponent是对patch过程的封装。patch的本质是将新旧vnode进行比较，创建、删除或者更新DOM节点/组件实例。

联系前后QA

Q：vue.js 同时多个赋值是一次性渲染还是多次渲染DOM？

A：官网已给出答案：[cn.vuejs.org/v2/guide/re…](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Freactivity.html)

> 可能你还没有注意到，Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。

> 例如，当你设置 vm.someData = 'new value'，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。多数情况我们不需要关心这个过程，但是如果你想基于更新后的 DOM 状态来做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员使用“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们必须要这么做。为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 Vue.nextTick(callback)。这样回调函数将在 DOM 更新完成后被调用。

这样是不是有种前后连贯起来的感觉，原来 nextTick 是这样子的。

- [参考link1](https://juejin.cn/post/6844904004388913160#heading-4)
- [参考link2](https://juejin.cn/post/6844904070981877774#heading-4)

### 第 6566 行至第 7069 行

- directives // 官网：[cn.vuejs.org/v2/guide/cu…](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcustom-directive.html)
- updateDirectives // 更新指令
- _update
- normalizeDirectives // 统一directives的格式
- getRawDirName // 返回指令名称 或者属性name名称+修饰符
- callHook$1 //触发指令钩子函数
- updateAttrs // 更新属性
- setAttr // 设置属性
- baseSetAttr
- updateClass // 更新样式
- klass
- parseFilters // 处理value 解析成正确的value，把过滤器 转换成 vue 虚拟dom的解析方法函数 比如把过滤器 ' ab | c | d' 转换成 _f("d")(_f("c")(ab))
- wrapFilter // 转换过滤器格式
- baseWarn // 基础警告
- pluckModuleFunction //循环过滤数组或者对象的值，根据key循环 过滤对象或者数组[key]值，如果不存在则丢弃，如果有相同多个的key值，返回多个值的数组
- addProp //在虚拟dom中添加prop属性
- addAttr //添加attrs属性
- addRawAttr //添加原始attr(在预转换中使用)
- addDirective //为虚拟dom 添加一个 指令directives属性 对象
- addHandler // 为虚拟dom添加events 事件对象属性

前面围绕“指令”和“过滤器”的一些基础工具函数。

后面围绕为虚拟 dom 添加属性、事件等具体实现函数。

### 第 7071 行至第 7298 行

- getRawBindingAttr
- getBindingAttr //  获取 :属性 或者v-bind:属性，或者获取属性 移除传进来的属性name，并且返回获取到 属性的值
- getAndRemoveAttr // 移除传进来的属性name，并且返回获取到 属性的值
- getAndRemoveAttrByRegex
- rangeSetItem
- genComponentModel // 为虚拟dom添加model属性

```arduino
    /*
    * Parse a v-model expression into a base path and a final key segment.
    * Handles both dot-path and possible square brackets.
    * 将 v-model 表达式解析为基路径和最后一个键段。
    * 处理点路径和可能的方括号。
    */
复制代码
```

- parseModel //转义字符串对象拆分字符串对象  把后一位key分离出来

// 如果数据是object.info.name的情况下 则返回是 {exp: "object.info",key: "name"} // 如果数据是object[info][name]的情况下 则返回是 {exp: "object[info]",key: "name"}

- next
- eof
- parseBracket //检测 匹配[] 一对这样的=括号
- parseString // 循环匹配一对''或者""符号

这一部分包括：原生指令 v-bind 和为虚拟 dom 添加 model 属性，以及格式校验工具函数。

### 第 7300 行至第 7473 行

- model
- genCheckboxModel // 为input type="checkbox" 虚拟dom添加 change 函数 ，根据v-model是否是数组，调用change函数，调用 set 去更新 checked选中数据的值
- genRadioModel // 为虚拟dom  inpu标签 type === 'radio' 添加change 事件 更新值
- genSelect // 为虚拟dom添加change 函数 ，change 函数调用 set 去更新 select 选中数据的值
- genDefaultModel //  如果虚拟dom标签是  'input' 类型不是checkbox，radio 或者是'textarea' 标签的时候，获取真实的dom的value值调用 change或者input方法执行set方法更新数据

[参考link](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fhao123456%2Fp%2F10616356.html)

### 阶段小结

- v-bind、v-model

区别：

1. v-bind 用来绑定数据和属性以及表达式，缩写为'：'
2. v-model 使用在表单中，实现双向数据绑定的，在表单元素外使用不起作用

Q：你知道v-model的原理吗？说说看

A: v-model本质上是语法糖，即利用v-model绑定数据，其实就是既绑定了数据，又添加了一个input事件监听 [link](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhaizlin%2Ffe-interview%2Fissues%2F560)

- 自定义指令钩子函数

一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

```markdown
1. bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
2. inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
3. update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
4. componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
5. unbind：只调用一次，指令与元素解绑时调用。
复制代码
```

- 指令钩子函数会被传入以下参数：

```perl
1. el：指令所绑定的元素，可以用来直接操作 DOM 。
2. binding：一个对象，包含以下属性：
     name：指令名，不包括 v- 前缀。
     value：指令的绑定值，例如：v-my-directive="1 + 1" 中，绑定值为 2。
     oldValue：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
     expression：字符串形式的指令表达式。例如 v-my-directive="1 + 1" 中，表达式为 "1 + 1"。
     arg：传给指令的参数，可选。例如 v-my-directive:foo 中，参数为 "foo"。
     modifiers：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。
3. vnode：Vue 编译生成的虚拟节点。移步 VNode API 来了解更多详情。
4. oldVnode：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。
复制代码
```

除了 el 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 dataset 来进行。

[【译】vue 自定义指令的魅力](https://juejin.cn/post/6844903508785758215)

### 第 7473 行至第 7697 行

- normalizeEvents // 为事件 多添加 change 或者input 事件加进去
- createOnceHandler$1
- add$1 // 为真实的dom添加事件
- remove$2
- updateDOMListeners // 更新dom事件
- updateDOMProps // 更新真实dom的props属性
- shouldUpdateValue // 判断是否需要更新value
- isNotInFocusAndDirty
- isDirtyWithModifiers // 判断脏数据修改 [脏数据概念](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2FLVXIANGAN%2Farticle%2Fdetails%2F85329630)

### 第 7699 行至第 7797 行

- domProps
- parseStyleText // 把style 字符串 转换成对象
- normalizeStyleData // 在同一个vnode上合并静态和动态样式数据
- normalizeStyleBinding // 将可能的数组/字符串值规范化为对象
- getStyle

```arduino
    /**
    * parent component style should be after child's
    * so that parent component's style could override it
    * 父组件样式应该在子组件样式之后
    * 这样父组件的样式就可以覆盖它
    * 循环子组件和组件的样式，把它全部合并到一个样式对象中返回 样式对象 如{width:100px,height:200px} 返回该字符串。
    */
复制代码
```

- setProp // 设置 prop

### 第 7799 行至第 7995 行

- normalize  // 给css加前缀。解决浏览器兼用性问题，加前缀
- updateStyle // 将vonde虚拟dom的css 转义成并且渲染到真实dom的csszhong
- addClass // 为真实dom 元素添加class类
- removeClass // 删除真实dom的css类名
- resolveTransition // 解析vonde中的transition的name属性获取到一个css过度对象类
- autoCssTransition // 通过 name 属性获取过渡 CSS 类名   比如标签上面定义name是 fade  css就要定义  .fade-enter-active,.fade-leave-active，.fade-enter,.fade-leave-to 这样的class
- nextFrame // 下一帧

### 第 7997 行至第 8093 行

- addTransitionClass // 获取 真实dom addTransitionClass 记录calss类
- removeTransitionClass // 删除vonde的class类和删除真实dom的class类
- whenTransitionEnds // 获取动画的信息，执行动画。
- getTransitionInfo // 获取transition，或者animation 动画的类型，动画个数，动画执行时间

这一部分关于：对真实 dom 的操作，包括样式的增删、事件的增删、动画类等。

回过头再理一下宏观上的东西，再来亿遍-虚拟DOM：模板 → 渲染函数 → 虚拟DOM树 → 真实DOM



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/1732381297ec7d20~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



那么这一部分则处在“虚拟DOM树 → 真实DOM”这个阶段

### 第 8093 行至第 8518 行

- getTimeout

```less
// Old versions of Chromium (below 61.0.3163.100) formats floating pointer numbers
// in a locale-dependent way, using a comma instead of a dot.
// If comma is not replaced with a dot, the input will be rounded down (i.e. acting
// as a floor function) causing unexpected behaviors

// 根据本地的依赖方式，Chromium 的旧版本（低于61.0.3163.100）格式化浮点数字，使用逗号而不是点。如果逗号未用点代替，则输入将被四舍五入而导致意外行为
复制代码
```

- toMs // [update toMs function. fix #4894](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fpull%2F8495%2Ffiles)
- enter

```css
// activeInstance will always be the <transition> component managing this
// transition. One edge case to check is when the <transition> is placed
// as the root node of a child component. In that case we need to check
// <transition>'s parent for appear check.

// activeInstance 将一直作为<transition>的组件来管理 transition。要检查的一种边缘情况：<transition> 作为子组件的根节点时。在这种情况下，我们需要检查 <transition> 的父项的展现。
复制代码
```

- leave // 离开动画
- performLeave
- checkDuration // only used in dev mode : 检测 val 必需是数字
- isValidDuration
- getHookArgumentsLength // 检测钩子函数 fns 的长度
- _enter
- createPatchFunction // path 把vonde 渲染成真实的dom：创建虚拟 dom - 函数体在 5845 行
- directive // 生命指令：包括 插入 和 组件更新

> 更新指令 比较 oldVnode 和 vnode，根据oldVnode和vnode的情况 触发指令钩子函数bind，update，inserted，insert，componentUpdated，unbind钩子函数

此节前部分是 transition 动画相关工具函数，后部分关于虚拟 Dom patch、指令的更新。

### 第 8520 行至第 8584 行

- setSelected // 设置选择 - 指令更新的工具函数
- actuallySetSelected // 实际选择，在 setSelected() 里调用
- hasNoMatchingOption // 没有匹配项 - 指令组件更新工具函数
- getValue // 获取 option.value
- onCompositionStart // 组成开始 - 指令插入工具函数
- onCompositionEnd // 组成结束-指令插入工具函数：防止无故触发输入事件
- trigger // 触发事件

### 第 8592 行至第 8728 行

// 定义在组件根内部递归搜索可能存在的 transition

- locateNode
- show // 控制 el 的 display 属性
- platformDirectives // 平台指令
- transitionProps // 过渡Props对象

```arduino
    // in case the child is also an abstract component, e.g. <keep-alive>
    // we want to recursively retrieve the real component to be rendered
    // 如果子对象也是抽象组件，例如<keep-alive>
    // 我们要递归地检索要渲染的实际组件
复制代码
```

- getRealChild
- extractTransitionData // 提取 TransitionData
- placeholder // 占位提示
- hasParentTransition // 判断是否有 ParentTransition
- isSameChild // 判断子对象是否相同

### 第 8730 行至第 9020 行

- Transition // !important

前部分以及此部分大部分围绕 Transition 这个关键对象。即迎合官网 “过渡 & 动画” 这一节，是我们需要关注的重点！

> Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。包括以下工具：
>
> - 在 CSS 过渡和动画中自动应用 class
> - 可以配合使用第三方 CSS 动画库，如 Animate.css
> - 在过渡钩子函数中使用 JavaScript 直接操作 DOM
> - 可以配合使用第三方 JavaScript 动画库，如 Velocity.js
>
> 在这里，我们只会讲到进入、离开和列表的过渡，你也可以看下一节的[管理过渡状态](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Ftransitioning-state.html)。

vue - transition 里面大有东西，这里有一篇[“细谈”](https://juejin.cn/post/6844903858611683336)推荐阅读。

- props
- TransitionGroup // TransitionGroup
- callPendingCbs // Pending 回调
- recordPosition // 记录位置
- applyTranslation // 应用动画 - TransitionGroup.updated 调用

```less
// we divide the work into three loops to avoid mixing DOM reads and writes
// in each iteration - which helps prevent layout thrashing.

//我们将工作分为三个 loops，以避免将 DOM 读取和写入混合在一起
//在每次迭代中-有助于防止布局冲撞。
复制代码
```

- platformComponents // 平台组件

```scss
// 安装平台运行时指令和组件
extend(Vue.options.directives, platformDirectives);
extend(Vue.options.components, platformComponents);
复制代码
```

Q: vue自带的内置组件有什么？

A: Vue中内置的组件有以下几种：

1. component

component组件：有两个属性---is inline-template

渲染一个‘元组件’为动态组件，按照'is'特性的值来渲染成那个组件

1. transition

transition组件：为组件的载入和切换提供动画效果，具有非常强的可定制性，支持16个属性和12个事件

1. transition-group

transition-group：作为多个元素/组件的过渡效果

1. keep-alive

keep-alive：包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们

1. slot

slot：作为组件模板之中的内容分发插槽，slot元素自身将被替换

### 第 9024 行至第 9207 行

// install platform specific utils // 安装平台特定的工具

- Vue.config.x

```ini
Vue.config.mustUseProp = mustUseProp;
Vue.config.isReservedTag = isReservedTag;
Vue.config.isReservedAttr = isReservedAttr;
Vue.config.getTagNamespace = getTagNamespace;
Vue.config.isUnknownElement = isUnknownElement;
复制代码
```

- Vue.prototype.$mount // public mount method 安装方法 实例方法挂载 vm

```javascript
// public mount method
Vue.prototype.$mount = function (
    el, // 真实dom 或者是 string
    hydrating //新的虚拟dom vonde
) {
    el = el && inBrowser ? query(el) : undefined;
    return mountComponent(this, el, hydrating)
};
复制代码
```

**devtools global hook** // 开发环境全局 hook Tip

- buildRegex // 构建的正则匹配
- parseText // 匹配view 指令，并且把他转换成 虚拟dom vonde 需要渲染的函数,比如指令{{name}}转换成 _s(name)
- transformNode // 获取 class 属性和:class或者v-bind的动态属性值，并且转化成字符串 添加到staticClass和classBinding 属性中
- genData // 初始化扩展指令 baseDirectives，on,bind,cloak方法，dataGenFns 获取到一个数组，数组中有两个函数 genData（转换 class） 和 genData$1（转换 style）,
- transformNode$1 // transformNode$1 获取 style属性和:style或者v-bind的动态属性值，并且转化成字符串 添加到staticStyle和styleBinding属性中
- genData$1 // 参见 genData
- style$1 // 包含 staticKeys、transformNode、genData 属性

### 第 9211 行至第 9537 行

- he
- isUnaryTag // 工具函数
- canBeLeftOpenTag // 工具函数
- isNonPhrasingTag // 工具函数 **Regular Expressions**
- parseHTML // 解析成 HTML !important

parseHTML 这个函数实现大概两百多行，是一个比较大的函数体了。

parseHTML 中的方法用于处理HTML开始和结束标签。

parseHTML 方法的整体逻辑是用正则判断各种情况，进行不同的处理。其中调用到了 options 中的自定义方法。

options 中的自定义方法用于处理AST语法树，最终返回出整个AST语法树对象。

贴一下源码，有兴趣可自行感受一二。附一篇详解[Vue.js HTML解析细节学习](https://link.juejin.cn?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000013592119)

```ini
function parseHTML(html, options) {
    var stack = [];
    var expectHTML = options.expectHTML;
    var isUnaryTag$$1 = options.isUnaryTag || no;
    var canBeLeftOpenTag$$1 = options.canBeLeftOpenTag || no;
    var index = 0;
    var last, lastTag;
    while (html) {
        last = html;
        // 确保我们不在像脚本/样式这样的纯文本内容元素中
        if (!lastTag || !isPlainTextElement(lastTag)) {
            var textEnd = html.indexOf('<');
            if (textEnd === 0) {
                // Comment:
                if (comment.test(html)) {
                    var commentEnd = html.indexOf('-->');

                    if (commentEnd >= 0) {
                        if (options.shouldKeepComment) {
                            options.comment(html.substring(4, commentEnd), index, index + commentEnd + 3);
                        }
                        advance(commentEnd + 3);
                        continue
                    }
                }

                // http://en.wikipedia.org/wiki/Conditional_comment#Downlevel-revealed_conditional_comment
                if (conditionalComment.test(html)) {
                    var conditionalEnd = html.indexOf(']>');

                    if (conditionalEnd >= 0) {
                        advance(conditionalEnd + 2);
                        continue
                    }
                }

                // Doctype:
                // 匹配 html 的头文件
                var doctypeMatch = html.match(doctype);
                if (doctypeMatch) {
                    advance(doctypeMatch[0].length);
                    continue
                }

                // End tag:
                var endTagMatch = html.match(endTag);
                if (endTagMatch) {
                    var curIndex = index;
                    advance(endTagMatch[0].length);
                    parseEndTag(endTagMatch[1], curIndex, index);
                    continue
                }

                // Start tag:
                // 解析开始标记
                var startTagMatch = parseStartTag();
                if (startTagMatch) {
                    handleStartTag(startTagMatch);
                    if (shouldIgnoreFirstNewline(startTagMatch.tagName, html)) {
                        advance(1);
                    }
                    continue
                }
            }

            var text = (void 0),
                rest = (void 0),
                next = (void 0);
            if (textEnd >= 0) {
                rest = html.slice(textEnd);
                while (
                    !endTag.test(rest) &&
                    !startTagOpen.test(rest) &&
                    !comment.test(rest) &&
                    !conditionalComment.test(rest)
                ) {
                    // < in plain text, be forgiving and treat it as text
                    next = rest.indexOf('<', 1);
                    if (next < 0) {
                        break
                    }
                    textEnd += next;
                    rest = html.slice(textEnd);
                }
                text = html.substring(0, textEnd);
            }

            if (textEnd < 0) {
                text = html;
            }

            if (text) {
                advance(text.length);
            }

            if (options.chars && text) {
                options.chars(text, index - text.length, index);
            }
        } else {
            //  处理是script,style,textarea
            var endTagLength = 0;
            var stackedTag = lastTag.toLowerCase();
            var reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'));
            var rest$1 = html.replace(reStackedTag, function (all, text, endTag) {
                endTagLength = endTag.length;
                if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') {
                    text = text
                        .replace(/<!\--([\s\S]*?)-->/g, '$1') // #7298
                        .replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1');
                }
                if (shouldIgnoreFirstNewline(stackedTag, text)) {
                    text = text.slice(1);
                }
                if (options.chars) {
                    options.chars(text);
                }
                return ''
            });
            index += html.length - rest$1.length;
            html = rest$1;
            parseEndTag(stackedTag, index - endTagLength, index);
        }

        if (html === last) {
            options.chars && options.chars(html);
            if (!stack.length && options.warn) {
                options.warn(("Mal-formatted tag at end of template: \"" + html + "\""), {
                    start: index + html.length
                });
            }
            break
        }
    }

    // Clean up any remaining tags
    parseEndTag();

    function advance(n) {
        index += n;
        html = html.substring(n);
    }

    function parseStartTag() {
        var start = html.match(startTagOpen);
        if (start) {
            var match = {
                tagName: start[1],
                attrs: [],
                start: index
            };
            advance(start[0].length);
            var end, attr;
            while (!(end = html.match(startTagClose)) && (attr = html.match(dynamicArgAttribute) || html.match(attribute))) {
                attr.start = index;
                advance(attr[0].length);
                attr.end = index;
                match.attrs.push(attr);
            }
            if (end) {
                match.unarySlash = end[1];
                advance(end[0].length);
                match.end = index;
                return match
            }
        }
    }

    function handleStartTag(match) {
        var tagName = match.tagName;
        var unarySlash = match.unarySlash;

        if (expectHTML) {
            if (lastTag === 'p' && isNonPhrasingTag(tagName)) {
                parseEndTag(lastTag);
            }
            if (canBeLeftOpenTag$$1(tagName) && lastTag === tagName) {
                parseEndTag(tagName);
            }
        }

        var unary = isUnaryTag$$1(tagName) || !!unarySlash;

        var l = match.attrs.length;
        var attrs = new Array(l);
        for (var i = 0; i < l; i++) {
            var args = match.attrs[i];
            var value = args[3] || args[4] || args[5] || '';
            var shouldDecodeNewlines = tagName === 'a' && args[1] === 'href' ?
                options.shouldDecodeNewlinesForHref :
                options.shouldDecodeNewlines;
            attrs[i] = {
                name: args[1],
                value: decodeAttr(value, shouldDecodeNewlines)
            };
            if (options.outputSourceRange) {
                attrs[i].start = args.start + args[0].match(/^\s*/).length;
                attrs[i].end = args.end;
            }
        }

        if (!unary) {
            stack.push({
                tag: tagName,
                lowerCasedTag: tagName.toLowerCase(),
                attrs: attrs,
                start: match.start,
                end: match.end
            });
            lastTag = tagName;
        }

        if (options.start) {
            options.start(tagName, attrs, unary, match.start, match.end);
        }
    }

    function parseEndTag(tagName, start, end) {
        var pos, lowerCasedTagName;
        if (start == null) {
            start = index;
        }
        if (end == null) {
            end = index;
        }

        // Find the closest opened tag of the same type
        if (tagName) {
            lowerCasedTagName = tagName.toLowerCase();
            for (pos = stack.length - 1; pos >= 0; pos--) {
                if (stack[pos].lowerCasedTag === lowerCasedTagName) {
                    break
                }
            }
        } else {
            // If no tag name is provided, clean shop
            pos = 0;
        }

        if (pos >= 0) {
            // Close all the open elements, up the stack
            for (var i = stack.length - 1; i >= pos; i--) {
                if (i > pos || !tagName &&
                    options.warn
                ) {
                    options.warn(
                        ("tag <" + (stack[i].tag) + "> has no matching end tag."), {
                            start: stack[i].start,
                            end: stack[i].end
                        }
                    );
                }
                if (options.end) {
                    options.end(stack[i].tag, start, end);
                }
            }

            // Remove the open elements from the stack
            stack.length = pos;
            lastTag = pos && stack[pos - 1].tag;
        } else if (lowerCasedTagName === 'br') {
            if (options.start) {
                options.start(tagName, [], true, start, end);
            }
        } else if (lowerCasedTagName === 'p') {
            if (options.start) {
                options.start(tagName, [], false, start, end);
            }
            if (options.end) {
                options.end(tagName, start, end);
            }
        }
    }
}
复制代码
```

### 第 9541 行至第 9914 行

**Regular Expressions** // 相关正则

- createASTElement // Convert HTML string to AST.
- parse // !important

parse 函数从 9593 行至 9914 行，共三百多行。核心吗？当然核心！

引自 wikipedia：

> 在计算机科学和语言学中，语法分析（英语：syntactic analysis，也叫 parsing）是根据某种给定的形式文法对由单词序列（如英语单词序列）构成的输入文本进行分析并确定其语法结构的一种过程。
>
> 语法分析器（parser）通常是作为编译器或解释器的组件出现的，它的作用是进行语法检查、并构建由输入的单词组成的数据结构（一般是语法分析树、抽象语法树等层次化的数据结构）。语法分析器通常使用一个独立的词法分析器从输入字符流中分离出一个个的“单词”，并将单词流作为其输入。实际开发中，语法分析器可以手工编写，也可以使用工具（半）自动生成。

**parse 的整体流程实际上就是先处理了一些传入的options，然后执行了parseHTML 函数，传入了template，options和相关钩子。**

具体实现这里盗一个图：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/17323819fa2394a6~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



parse中的语法分析可以看[这一篇这一节](https://juejin.cn/post/6844903871379144711#heading-13)

1. start
2. char
3. comment
4. end

parse、optimize、codegen的核心思想解读可以看[这一篇这一节](https://juejin.cn/post/6844903861975531528#heading-6)

这里实现的细节还真不少！

### 阶段小结（重点）

噫嘘唏！来到第 20 篇的小结！来个图镇一下先！

还记得官方这样的一句话吗？

> 下图展示了实例的生命周期。你不需要立马弄明白所有的东西，不过随着你的不断学习和使用，它的参考价值会越来越高。

看了这么多，我们再回头看看注释版。



![注释版](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/1732382028834939~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

[link](https://link.juejin.cn?target=http%3A%2F%2Fwww.shangdixinxi.com%2Fdetail-1105609.html)



上图值得一提的是：**Has "template" option?** 这个逻辑的细化

> 碰到是否有 template 选项时，会询问是否要对 template 进行编译：即模板通过编译生成 AST，再由 AST 生成 Vue 的渲染函数，渲染函数结合数据生成 Virtual DOM 树，对 Virtual DOM 进行 diff 和 patch 后生成新的UI。

如图（此图前文也有提到，见 0 至 5000 行总结）： 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/17323808ba0442e2~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



将 Vue 的源码的“数据监听”、“虚拟 DOM”、“Render 函数”、“组件编译”、结合好，则算是融会贯通了！

**一图胜万言**



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/17323830402969ff~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



好好把上面的三张图看懂，便能做到“成竹在胸”，走遍天下的 VUE 原理面试都不用慌了。框架就在这里，细化的东西就需要多多记忆了！

### 第 9916 行至第 10435 行

🙌 到 1w 行了，自我庆祝一下！

- processRawAttrs // parse 方法里用到的工具函数 用于将特性保存到AST对象的attrs属性上
- processElement// parse 方法工具函数 元素填充

```scss
export function processElement (
  element: ASTElement,
  options: CompilerOptions
) {
  processKey(element)

  // determine whether this is a plain element after
  // removing structural attributes
  element.plain = (
    !element.key &&
    !element.scopedSlots &&
    !element.attrsList.length
  )

  processRef(element)
  processSlotContent(element)
  processSlotOutlet(element)
  processComponent(element)
  for (let i = 0; i < transforms.length; i++) {
    element = transforms[i](element, options) || element
  }
  processAttrs(element)
  return element
}
复制代码
```

可以看到主要函数包括：processKey、processRef、processSlotContent、processSlotOutlet、processComponent、processAttrs 和最后遍历执行的transforms。

processElement完成的slotTarget的赋值，这里则是将所有的slot创建的astElement以对象的形式赋值给currentParent的scopedSlots。以便后期组件内部实例话的时候可以方便去使用vm.?slot。

- processKey
- processRef

1. 首先最为简单的是processKey和processRef,在这两个函数处理之前，我们的key属性和ref属性都是保存在astElement上面的attrs和attrsMap，经过这两个函数之后，attrs里面的key和ref会被干掉，变成astElement的直属属性。
2. 探讨一下slot的处理方式，我们知道的是，slot的具体位置是在组件中定义的，而需要替换的内容又是组件外面嵌套的代码，Vue对这两块的处理是分开的。

先说组件内的属性摘取，主要是slot标签的name属性，这是processSlotOutLet完成的。

- processFor
- parseFor
- processIf
- processIfConditions
- findPrevElement
- addIfCondition
- processOnce
- processSlotContent
- getSlotName
- processSlotOutlet

```csharp
// handle <slot/> outlets
function processSlotOutlet (el) {
  if (el.tag === 'slot') {
    el.slotName = getBindingAttr(el, 'name') // 就是这一句了。
    if (process.env.NODE_ENV !== 'production' && el.key) {
      warn(
        `\`key\` does not work on <slot> because slots are abstract outlets ` +
        `and can possibly expand into multiple elements. ` +
        `Use the key on a wrapping element instead.`,
        getRawBindingAttr(el, 'key')
      )
    }
  }
}
// 其次是摘取需要替换的内容，也就是 processSlotContent，这是是处理展示在组件内部的slot，但是在这个地方只是简单的将给el添加两个属性作用域插槽的slotScope和 slotTarget，也就是目标slot。
复制代码
```

- processComponent // processComponent 并不是处理component，而是摘取动态组件的is属性。 processAttrs是获取所有的属性和动态属性。
- processAttrs
- checkInFor
- parseModifiers
- makeAttrsMap

这一部分仍是衔接这 parse function 里的具体实现：start、end、comment、chars四大函数。

流程再回顾一下：

一、普通标签处理流程描述

1. 识别开始标签，生成匹配结构match。

const match = { // 匹配startTag的数据结构 tagName: 'div', attrs: [ { 'id="xxx"','id','=','xxx' }, ... ], start: index, end: xxx } 复制代码 2. 处理attrs，将数组处理成 {name:'xxx',value:'xxx'} 3. 生成astElement，处理for,if和once的标签。 4. 识别结束标签，将没有闭合标签的元素一起处理。 5. 建立父子关系，最后再对astElement做所有跟Vue 属性相关对处理。slot、component等等。

二、文本或表达式的处理流程描述。

1. 截取符号<之前的字符串，这里一定是所有的匹配规则都没有匹配上，只可能是文本了。
2. 使用chars函数处理该字符串。
3. 判断字符串是否含有delimiters，默认也就是${},有的话创建type为2的节点，否则type为3.

三、注释流程描述

1. 匹配注释符号。
2. 使用comment函数处理。
3. 直接创建type为3的节点。

[参考 link](https://juejin.cn/post/6844903869831446541)

### 阶段小结

parseHTML() 和 parse() 这两个函数占了很大的篇幅，值得重点去看看。的确也很多细节，一些正则的匹配，字符串的操作等。从宏观上把握从 template 到 vnode 的 parse 流程也无大问题。

### 第 10437 行至第 10605 行

- isTextTag // function chars() 里的工具函数
- isForbiddenTag //  function parseHTML() 用到的工具函数用于检查元素标签是否合法（不是保留命名）
- guardIESVGBug // parse start() 中用到的工具函数
- checkForAliasModel // checkForAliasModel用于检查v-model的参数是否是v-for的迭代对象
- preTransformNode // preTransformNode 方法对el进行预处理，便于后续对标签上的指令和属性进行处理，然后进行树结构的构建，确定el的root, parent, children等属性。总结下来就是生成树节点，构建树结构(关联树节点)。
- cloneASTElement // 转换属性，把数组属性转换成对象属性，返回对象 AST元素
- text // 为虚拟dom添加textContent 属性
- html // 为虚拟dom添加innerHTML 属性
- baseOptions

```swift
var baseOptions = {
  expectHTML: true, //标志 是html
  modules: modules$1, //为虚拟dom添加staticClass，classBinding，staticStyle，styleBinding，for，
                      //alias，iterator1，iterator2，addRawAttr ，type ，key， ref，slotName
                      //或者slotScope或者slot，component或者inlineTemplate ，plain，if ，else，elseif 属性
  directives: directives$1, //根据判断虚拟dom的标签类型是什么？给相应的标签绑定 相应的 v-model 双数据绑定代码函数，
                            //为虚拟dom添加textContent 属性，为虚拟dom添加innerHTML 属性
  isPreTag: isPreTag, // 判断标签是否是 pre
  isUnaryTag: isUnaryTag, // 匹配标签是否是area,base,br,col,embed,frame,hr,img,input,
                          // isindex,keygen, link,meta,param,source,track,wbr
  mustUseProp: mustUseProp,
  canBeLeftOpenTag: canBeLeftOpenTag,// 判断标签是否是 colgroup,dd,dt,li,options,p,td,tfoot,th,thead,tr,source
  isReservedTag: isReservedTag, // 保留标签 判断是不是真的是 html 原有的标签 或者svg标签
  getTagNamespace: getTagNamespace, // 判断 tag 是否是svg或者math 标签
  staticKeys: genStaticKeys(modules$1) // 把数组对象 [{ staticKeys:1},{staticKeys:2},{staticKeys:3}]连接数组对象中的 staticKeys key值，连接成一个字符串 str=‘1,2,3’
};
复制代码
```

- genStaticKeysCached

### 第 10607 行至第 10731 行

```less
/**
  * Goal of the optimizer: walk the generated template AST tree
  * and detect sub-trees that are purely static, i.e. parts of
  * the DOM that never needs to change.
  *
  * Once we detect these sub-trees, we can:
  *
  * 1. Hoist them into constants, so that we no longer need to
  *    create fresh nodes for them on each re-render;
  * 2. Completely skip them in the patching process.
  */
  // 优化器的目标:遍历生成的模板AST树检测纯静态的子树，即永远不需要更改的DOM。
  // 一旦我们检测到这些子树，我们可以:
  // 1。把它们变成常数，这样我们就不需要了
  // 在每次重新渲染时为它们创建新的节点;
  // 2。在修补过程中完全跳过它们。
复制代码
```

- optimize // !important:过 parse 过程后，会输出生成 AST 树，接下来需要对这颗树做优化。即这里的 optimize // 循环递归虚拟node，标记是不是静态节点 // 根据node.static或者 node.once 标记staticRoot的状态
- genStaticKeys$1
- markStatic$1 // 标准静态节点
- markStaticRoots // 标注静态根（重要）
- isStatic // isBuiltInTag（即tag为component 和slot）的节点不会被标注为静态节点，isPlatformReservedTag（即平台原生标签，web 端如 h1 、div标签等）也不会被标注为静态节点。
- isDirectChildOfTemplateFor

### 阶段小结

简单来说：整个 optimize 的过程实际上就干 2 件事情，markStatic(root) 标记静态节点 ，markStaticRoots(root, false) 标记静态根节点。

那么被判断为静态根节点的条件是什么？

1. 该节点的所有子孙节点都是静态节点（判断为静态节点要满足 7 个判断，[详见](https://juejin.cn/post/6844903910059016200)）
2. 必须存在子节点
3. 子节点不能只有一个纯文本节点

其实，markStaticRoots()方法针对的都是普通标签节点。表达式节点与纯文本节点都不在考虑范围内。

markStatic()得出的static属性，在该方法中用上了。将每个节点都判断了一遍static属性之后，就可以更快地确定静态根节点：通过判断对应节点是否是静态节点 且 内部有子元素 且 单一子节点的元素类型不是文本类型。

> 只有纯文本子节点时，他是静态节点，但不是静态根节点。静态根节点是 optimize 优化的条件，没有静态根节点，说明这部分不会被优化。

Q：为什么子节点的元素类型是静态文本类型，就会给 optimize 过程加大成本呢？

A：optimize 过程中做这个静态根节点的优化目是：在 patch 过程中，减少不必要的比对过程，加速更新。但是需要以下成本

1. 维护静态模板的存储对象 一开始的时候，所有的静态根节点 都会被解析生成 VNode，并且被存在一个缓存对象中，就在 Vue.proto._staticTree 中。 随着静态根节点的增加，这个存储对象也会越来越大，那么占用的内存就会越来越多 势必要减少一些不必要的存储，所有只有纯文本的静态根节点就被排除了
2. 多层render函数调用 这个过程涉及到实际操作更新的过程。在实际render 的过程中，针对静态节点的操作也需要调用对应的静态节点渲染函数，做一定的判断逻辑。这里需要一定的消耗。

纯文本直接对比即可，不进行 optimize 将会更高效。

[参考link](https://juejin.cn/post/6844904098605563911)

### 第 10733 行至第 10915 行

// KeyboardEvent.keyCode aliases

- keyCodes // 内置按键
- keyNames
- genGuard // genGuard = condition => `if(${condition})return null;`
- modifierCode //m odifierCode生成内置修饰符的处理
- genHandlers
- genHandler // 调用genHandler处理events[name]，events[name]可能是数组也可能是独立对象，取决于name是否有多个处理函数。
- genKeyFilter // genKeyFilter用于生成一段过滤的字符串：
- genFilterCode // 在 genKeyFilter 里被调用
- on
- bind$1
- baseDirectives // CodegenState 里的工具函数

不管是组件还是普通标签，事件处理代码都在genData的过程中，和之前分析原生事件一致，genHandlers用来处理事件对象并拼接成字符串。

### 第 10921 行至第 11460 行

// generate(ast, options)

```arduino
export function generate (
  ast: ASTElement | void,
  options: CompilerOptions
): CodegenResult {
  const state = new CodegenState(options)
  const code = ast ? genElement(ast, state) : '_c("div")'
  return {
    render: `with(this){return ${code}}`,
    staticRenderFns: state.staticRenderFns
  }
}
复制代码
```

- CodegenState
- generate // ！important
- genElement

```kotlin
export function genElement (el: ASTElement, 
state: CodegenState): string {
  if (el.parent) {
    el.pre = el.pre || el.parent.pre
  }

  if (el.staticRoot && !el.staticProcessed) {
    // 如果是一个静态的树， 如 <div id="app">123</div>
    // 生成_m()方法
    // 静态的渲染函数被保存至staticRenderFns属性中
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    // v-once 转化为_o()方法
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    // _l()
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    // v-if 会转换为表达式
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
    // 如果是template，处理子节点
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    // 如果是插槽，处理slot
    return genSlot(el, state)
  } else {
    // component or element
    let code
    // 如果是组件，处理组件
    if (el.component) {
      code = genComponent(el.component, el, state)
    } else {
      let data
      if (!el.plain || (el.pre && state.maybeComponent(el))) {
        data = genData(el, state)
      }

      const children = el.inlineTemplate ? null : genChildren(el, state, true)
      code = `_c('${el.tag}'${
        data ? `,${data}` : '' // data
      }${
        children ? `,${children}` : '' // children
      })`
    }
    // module transforms
    for (let i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code)
    }
    return code
  }
}

复制代码
```

- genStatic // genStatic会将ast转化为_m()方法
- genOnce // 如果v-once在v-for中，那么就会生成_o()方法， 否则将其视为静态节点
- genIf // genIf会将v-if转换为表达式，示例如下
- genIfConditions
- genFor // v-for会转换为_l()
- genData$2
- genDirectives // genData() 里调用
- genInlineTemplate // genData() 里调用
- genScopedSlots // genData() 里调用
- genScopedSlot
- genChildren // 处理子节点
- getNormalizationType // 用于判断是否需要规范化
- genNode // 处理 Node
- genText // 处理 Text
- genComment
- genSlot // 处理插槽
- genComponent // 处理组件
- genProps // 处理 props
- transformSpecialNewlines

这里面的逻辑、细节太多了，不做赘述，有兴趣了解的童鞋可以去看[推荐阅读](https://juejin.cn/post/6844903973510447117)

### 阶段小结

generate方法内部逻辑还是很复杂的，**但仅做了一件事情，就是将ast转化为render函数的字符串，形成一个嵌套结构的方法，模版编译生成的_c(),_m(),_l等等其实都是生成vnode的方法**，在执行vue.$mount方法的时候，会调用vm._update(vm._render(), hydrating)方法，此时_render()中方法会执行生成的render()函数，执行后会生成vnode，也就是虚拟dom节点。

### 第 11466 行至第 11965 行

- prohibitedKeywordRE // 正则校验：禁止关键字
- unaryOperatorsRE // 正则校验：一元表达式操作
- stripStringRE // 正则校验：脚本字符串
- detectErrors // 检测错误工具函数
- checkNode // 检查 Node
- checkEvent // 检查 Event
- checkFor // 检查 For 循环
- checkIdentifier // 检查 Identifier
- checkExpression // 检查表达式
- checkFunctionParameterExpression // 检查函数表达式
- generateCodeFrame
- repeat$1
- createFunction // 构建函数
- createCompileToFunctionFn // 构建编译函数
- compile // !important

```ini
return function createCompiler (baseOptions) {
function compile (
    template,
    options
) {
    var finalOptions = Object.create(baseOptions);
    var errors = [];
    var tips = [];

    var warn = function (msg, range, tip) {
    (tip ? tips : errors).push(msg);
    };

    if (options) {
    if (options.outputSourceRange) {
        // $flow-disable-line
        var leadingSpaceLength = template.match(/^\s*/)[0].length;

        warn = function (msg, range, tip) {
        var data = { msg: msg };
        if (range) {
            if (range.start != null) {
            data.start = range.start + leadingSpaceLength;
            }
            if (range.end != null) {
            data.end = range.end + leadingSpaceLength;
            }
        }
        (tip ? tips : errors).push(data);
        };
    }
    // merge custom modules
    if (options.modules) {
        finalOptions.modules =
        (baseOptions.modules || []).concat(options.modules);
    }
    // merge custom directives
    if (options.directives) {
        finalOptions.directives = extend(
        Object.create(baseOptions.directives || null),
        options.directives
        );
    }
    // copy other options
    for (var key in options) {
        if (key !== 'modules' && key !== 'directives') {
        finalOptions[key] = options[key];
        }
    }
    }

    finalOptions.warn = warn;

    var compiled = baseCompile(template.trim(), finalOptions);
    {
    detectErrors(compiled.ast, warn);
    }
    compiled.errors = errors;
    compiled.tips = tips;
    return compiled
}
复制代码
```

再看这张图，对于“模板编译”是不是有一种新的感觉了。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/17323830402969ff~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)



- compileToFunctions

// 最后的最后

```kotlin
    return Vue;
复制代码
```

哇！历时一个月左右，我终于完成啦！！！

完结撒花🎉🎉🎉！激动 + 释然 + 感恩 + 小满足 + ...... ✿✿ヽ(°▽°)ノ✿

这生啃给我牙齿都啃酸了！！

## 总结

emmm，本来打算再多修补一下，但是看到 vue3 的源码解析已有版本出来啦（扶朕起来，朕还能学），时不我待，Vue3 奥利给，干就完了！

后续仍会完善此文，您的点赞是我最大的动力！也望大家不吝赐教，不吝赞美~

最最最最后，还是那句老话，与君共勉：

> 纸上得来终觉浅 绝知此事要躬行



作者：掘金安东尼
链接：https://juejin.cn/post/6846687602679119885
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

在开始之前,阅读源码你需要有扎实的基本功,还要有耐心,能把握全局,不要扣细节!

## 一、先看看目录







```sql
├── build --------------------------------- 构建相关的文件
├── dist ---------------------------------- 构建后文件的输出目录
├── examples ------------------------------ 存放使用Vue开发的的例子
├── flow ---------------------------------- 类型声明，使用开源项目 [Flow](https://flowtype.org/)
├── package.json -------------------------- 项目依赖
├── test ---------------------------------- 包含所有测试文件
├── src ----------------------------------- 这个是我们最应该关注的目录，包含了源码
│   ├──platforms --------------------------- 包含平台相关的代码
│   │   ├──web -----------------------------  包含了不同构建的包的入口文件
│   │   |   ├──entry-runtime.js ---------------- 运行时构建的入口，输出 dist/vue.common.js 文件，不包含模板(template)到render函数的编译器，所以不支持 `template` 选项，我们使用vue默认导出的就是这个运行时的版本。大家使用的时候要注意
│   │   |   ├── entry-runtime-with-compiler.js -- 独立构建版本的入口，输出 dist/vue.js，它包含模板(template)到render函数的编译器
│   ├── compiler -------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
│   │   ├── parser ------------------------ 存放将模板字符串转换成元素抽象语法树的代码
│   │   ├── codegen ----------------------- 存放从抽象语法树(AST)生成render函数的代码
│   │   ├── optimizer.js ------------------ 分析静态树，优化vdom渲染
│   ├── core ------------------------------ 存放通用的，平台无关的代码
│   │   ├── observer ---------------------- 反应系统，包含数据观测的核心代码
│   │   ├── vdom -------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
│   │   ├── instance ---------------------- 包含Vue构造函数设计相关的代码
│   │   ├── global-api -------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── components -------------------- 包含抽象出来的通用组件
│   ├── server ---------------------------- 包含服务端渲染(server-side rendering)的相关代码
│   ├── sfc ------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
│   ├── shared ---------------------------- 包含整个代码库通用的代码复制代码
```

## 二、Vue 的构造函数是什么样的

使用 `new` 操作符来调用 `Vue`，`Vue`是一个构造函数,了解了目录结构,下面来看下下入口文件

打开package.json![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/20/16241cb0caf116e4~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

当我们运行npm run dev,看看干了啥,rollup也是类似webpack的打包工具,根据

TARGET=web-full-dev



去build/config.js查找

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/20/16241d00c8f27df5~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)



打开入口文件找到了`web/entry-runtime-with-compiler.js `

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/4/16329e8b592b1198~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

依照以上查找路径,我们找到了Vue构造函数

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/20/16241d9039ff50e3~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

定义了构造函数,引入依赖,调用初始化函数,最后导出Vue

```scss
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue复制代码
```

打开这五个文件，找到相应的方法，你会发现，这些方法的作用，就是在 Vue 的原型 prototype 上挂载方法或属性

### 1. 先进入 initMixin(Vue),在prototype上挂载

### `Vue.prototype._init = function (options) {}  复制代码`

### 2. 进入 stateMixin(Vue),在prototype上挂载 

### `Vue.prototype.$data  Vue.prototype.$props  Vue.prototype.$set = set  Vue.prototype.$delete = del  Vue.prototype.$watch = function(){}  复制代码`

### 3. 进入eventsMixin(Vue),在prototype上挂载 







```bash
Vue.prototype.$on 
Vue.prototype.$once 
Vue.prototype.$off 
Vue.prototype.$emit
复制代码
```



### 4.进入lifecycleMixin(Vue),在prototype上挂载 







```javascript
Vue.prototype._update 
Vue.prototype.$forceUpdate 
Vue.prototype.$destroy  复制代码
```

### 5. 最后进入renderMixin(Vue),在prototype上挂载 







```ini
Vue.prototype.$nextTick 
Vue.prototype._render 
Vue.prototype._o = markOnce 
Vue.prototype._n = toNumber 
Vue.prototype._s = toString 
Vue.prototype._l = renderList 
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual 
Vue.prototype._i = looseIndexOf 
Vue.prototype._m = renderStatic 
Vue.prototype._f = resolveFilter 
Vue.prototype._k = checkKeyCodes 
Vue.prototype._b = bindObjectProps 
Vue.prototype._v = createTextVNode 
Vue.prototype._e = createEmptyVNode 
Vue.prototype._u = resolveScopedSlots 
Vue.prototype._g = bindObjectListeners
复制代码
```



### 根据上面的查找路径下一步到src/core/index.js

引入依赖,在Vue上挂载静态方法和属性

1. 

   

   

   ```javascript
   import { initGlobalAPI } from './global-api/index'
   import { isServerRendering } from 'core/util/env'
   
   initGlobalAPI(Vue)
   
   Object.defineProperty(Vue.prototype, '$isServer', {
     get: isServerRendering
   })
   
   Object.defineProperty(Vue.prototype, '$ssrContext', {
     get () {
       /* istanbul ignore next */
       return this.$vnode && this.$vnode.ssrContext
     }
   })
   
   Vue.version = '__VERSION__'
   
   export default Vue复制代码
   ```

### 进入initGlobalAPI(Vue),在Vue上挂载静态属性和方法







```ini
Vue.config Vue.util = util
Vue.set = set 
Vue.delete = del 
Vue.nextTick = util.nextTick 
Vue.options = { 
 components: { KeepAlive }, 
 directives: {}, 
 filters: {}, 
 _base: Vue 
} 
Vue.use 
Vue.mixin 
Vue.cid = 0 
Vue.extend 
Vue.component = function(){} 
Vue.directive = function(){} 
Vue.filter = function(){} 
复制代码
```



### 接着挂载







```ini
Vue.prototype.$isServer 
Vue.version = '__VERSION__'
复制代码
```



###  



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/3/20/16241f784daf7aea~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

### 根据上面的查找路径下一步到runtime/index.js,安装平台特有的工具







```ini
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement
// 安装平台特定的 指令 和 组件
Vue.options = {
    components: {
        KeepAlive,
        Transition,
        TransitionGroup
    },
    directives: {
        model,
        show
    },
    filters: {},
    _base: Vue
}
Vue.prototype.__patch__
Vue.prototype.$mount复制代码
```



### 根据上面的查找路径最后一步到web/entry-runtime-with-compiler.js 

1. 缓存来自 `web-runtime.js` 文件的 `$mount` 函数,const mount = Vue.prototype.$mount
2. 在 Vue 上挂载 `compile`然后覆盖覆盖了 `Vue.prototype.$mount`
3. Vue.compile = compileToFunctions 就是将模板 `template` 编译为render函数。

**到这整个Vue构造函数就还原了**



## 三、通过一个例子说明整个流程

index.html







```xml
 <!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Vue.js grid component example</title>
    </head>
  <body>
    <div id="app">
        <ol>
          <li v-for="todo in todos">
            {{ todo.text }}
          </li>
        </ol>
        <Child></Child>
    </div>
  </body>
</html>
复制代码
```



grid.js







```css
let Child = {
	data: function() {
	  return {child: '你好哈'}
	},
	template: '<div>{{child}}</div>'
}
new Vue({
	el: '#app',
	data: {
		todos: [
                    {text: '学习 JavaScript'}, 
                    {text: '学习 Vue'}, 
                    {text: '整个牛项目'}]
	},
	components: {'Child': Child}
})复制代码
```

### 想要知道Vue都干了什么,首先查看构造函数function Vue (options) {  this._init(options) } 



- new Vue({*//传入上面的内容* }) ,首先进入Vue.prototype._init,构造函数第一个挂载的方法









```scss
Vue.prototype._init = function (options) {
    const vm= this
    vm._uid = uid++
    let startTag, endTag
    vm._isVue = true

    if (options && options._isComponent) {
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }

    vm._renderProxy = vm
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) 
    initState(vm)
    initProvide(vm)
    callHook(vm, 'created')
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
复制代码
```



`_init()` 方法在一开始的时候，在 `this` 对象上定义了两个属性：`_uid` 和 `_isVue`，然后判断有没有定义 `options._isComponent`这里会走 `else` 分支，也就是这段代码：







```ini
vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )mergeOptions使用策略模式合并传入的options和Vue.options复制代码
```

### mergeOptions使用策略模式合并传入的options和Vue.options合并后的代码结构, 可以看到通过合并策略components,directives,filters继承了全局的, 这就是为什么全局注册的可以在任何地方使用,因为每个实例都继承了全局的, 所以都能找到 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/4/1632a280202cac9f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

1. 

   接着调用了`initLifecycle``initEvents``initRender、initState``initState``beforeCreate``created,``created`**重点看下initState()**







```scss
initState (vm) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}复制代码
```

## 通过initData看Vue的数据响应系统,由于只是传了data,所以执行initData(vm)







```ini
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
      proxy(vm, `_data`, key)
  }
  observe(data, true /* asRootData */)
}
复制代码
```









```kotlin
proxy (target, sourceKey, key) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}复制代码
```



### 取出data中的key,进行循环,通过proxy代理,可以直接通过 this.属性 访问data中的值,在实例对象上对数据进行代理，这样我们就能通过this.todos 来访问 data.todos 了 接下啦,正式的数据响应系统observe(data, true /* asRootData */),将数据通过Object.defineProperty进行get,set处理,使得数据响应







```javascript
class Observer {
  constructor(value) {
    this.value = value 
    this.dep = new Dep() 
    this.vmCount = 0 
    def(value, '__ob__', this) 
    if (Array.isArray(value)) {
      const augment = hasProto ? protoAugment : copyAugment augment(value, arrayMethods, arrayKeys) this.observeArray(value)
    } else {
      this.walk(value)
    }
  }
  walk(obj: Object) {
    const keys = Object.keys(obj) for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }
  observeArray(items) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
} 
复制代码
```



**在** `**Observer**` **类中，我们使用** `**walk**` **方法对数据data的属性循环调用** `**defineReactive**` **方法，**`**defineReactive**` **方法很简单，仅仅是将数据data的属性转为访问器属性，并对数据进行递归观测，否则只能观测数据data的直属子属性。这样我们的第一步工作就完成了，当我们修改或者获取data属性值的时候，通过** `**get**` **和** `**set**` **即能获取到通知。**







```scss
function defineReactive (
  obj,
  key,
  val,
  customSetter,
  shallow
) {
  const dep = new Dep()
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      val = newVal
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
复制代码
```



1. 其中 **let childOb = !shallow && observe(val),进行递归调用,将所有data数据包含子集进行get set化,进行响应**

2. 其中在Observe类中,如果属性是数组,会进行改造

   

   

   

   ```javascript
   if (Array.isArray(value)) {
         const augment = hasProto
           ? protoAugment
           : copyAugment
         augment(value, arrayMethods, arrayKeys)
         this.observeArray(value)
       } else {
         this.walk(value)
       }export const arrayMethods = Object.create(arrayProto)
   ;[
     'push',
     'pop',
     'shift',
     'unshift',
     'splice',
     'sort',
     'reverse'
   ]
   .forEach(function (method) {
     const original = arrayProto[method]
     def(arrayMethods, method, function mutator (...args) {
       const result = original.apply(this, args)
       const ob = this.__ob__
       let inserted
       switch (method) {
         case 'push':
         case 'unshift':
           inserted = args
           break
         case 'splice':
           inserted = args.slice(2)
           break
       }
       if (inserted) ob.observeArray(inserted)
       // notify change
       ob.dep.notify()
       return result
     })
   })
   复制代码
   ```

   

**重写数组中的上述方法,当在对数组进行这些操作时,**ob.dep.notify(),通知相应的改变

### 有了模板经过compileToFunctions,将模板编译为ast语法树,经过静态优化, 最后处理成render函数,本例中的render函数,使用了with(this), 将this作用域提前,                              {{ todo.text }}                   所以我们可以直接在模板中使用属性,不加this! 当然加上this也是可以的 解除了我的很多迷惑,为什么模板中可以不加this(react中的render是要使用this的), 其中的v-for转换成_l,根据之前的Vue.prototype._l = renderList







```javascript
function() {
    with (this) {
        return _c('div', {
            attrs: {
                "id": "app"
            }
        }, [_c('ol', _l((this.todos), function(todo) {
            return _c('li', [_v("\n            " + _s(todo.text) + "\n          ")])
        })), _v(" "), _c('child')], 1)
    }
}复制代码
```



- 生成了render函数后接着进入mountComponent,

- 首先调用了

  beforeMount函数,

- 接着执行vm._watcher = new Watcher(vm, updateComponent, noop)

- 最后callHook(vm, 'mounted'),执行mounted,所以在mounted执行之前就已经挂载到dom上

### 所以重点 vm._watcher = new Watcher(vm, updateComponent, noop)







```ini
function mountComponent (
  vm,
  el,
  hydrating
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
  }
  callHook(vm, 'beforeMount')

  let updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }

  vm._watcher = new Watcher(vm, updateComponent, noop)
  hydrating = false
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
复制代码
```



**查看Watcher代码**









```kotlin
class Watcher {
  constructor (
    vm,
    expOrFn,
    cb,
    options
  ) {
    this.vm = vm
    vm._watchers.push(this)

    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    value = this.getter.call(vm, vm)
    return value
  }
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }
}
复制代码
```



执行构造函数由于 this.lazy=false; this.value = this.lazy      ? undefined      : this.get();

执行get方法 其中pushTarget(this),给Dep.target添加静态属性this(当前new Watcher()实例 )

 function pushTarget (_target) { 

 if (Dep.target) targetStack.push(Dep.target) 

 Dep.target = _target

 }







```kotlin
get () {
    pushTarget(this)
    let value
    const vm = this.vm
    value = this.getter.call(vm, vm)
    return value
  }
复制代码
```



```kotlin
接着执行 this.getter.call(vm, vm)复制代码
this.getter就是
updateComponent = () => {
vm._update(vm._render(), hydrating)
}      复制代码
```

1. 开始先调用vm._render()







```javascript
  Vue.prototype._render = function (){
    const vm = this
    const {
      render,
      staticRenderFns,
      _parentVnode
    } = vm.$options

    let vnode = render.call(vm._renderProxy, vm.$createElement)
    return vnode
  }
复制代码
```



开始执行之前编译好的render函数了,在执行render函数时,通过获取todos属性等,触发相应







```javascript
function() {
    with (this) {
        return _c('div', {
            attrs: {
                "id": "app"
            }
        }, [_c('ol', _l((this.todos), function(todo) {
            return _c('li', [_v("\n            " + _s(todo.text) + "\n          ")])
        })), _v(" "), _c('child')], 1)
    }
}复制代码
```



的get方法,这个时候Dep.target已经存在静态属性,Watcher实例了

所以相应的dep实例就会收集对应的Watcher实例了

```
复制代码
```

执行完之后返回vnode,

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/4/1632a8f207cfe250~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)



```bash
updateComponent = () => {
vm._update(vm._render(), hydrating)
} 
其中vm._render()执行render函数返回vnode作为参数
接下来执行vm._update
这是首次渲染,所以执行
vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false,
        vm.$options._parentElm,
        vm.$options._refElm
      )复制代码
```







```ini
Vue.prototype._update = function (vnode, hydrating) {
    const vm: Component = this
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    vm._vnode = vnode
    if (!prevVnode) {
      vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false,
        vm.$options._parentElm,
        vm.$options._refElm
      )
      vm.$options._parentElm = vm.$options._refElm = null
    } else {
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
  }
复制代码
```



### vm.__patch__(        vm.$el, vnode, hydrating, false,        vm.$options._parentElm,        vm.$options._refElm      ) 根据vnode中树,创建对应元素,插入到父节点中,通过对vnode递归循环创建所有子节点 插入到父节点中 其中如果遇到子元素是组件,例如本例中Child,会创建对应VueComponent的实例,执行 和new Vue()一样的流程 

如果还没有 `prevVnode` 说明是首次渲染，直接创建真实DOM。如果已经有了 `prevVnode` 说明不是首次渲染，那么就采用 `patch` 算法进行必要的DOM操作。这就是Vue更新DOM的逻辑。只不过我们没有将 virtual DOM 内部的实现。

当改变属性值时,会触发对应的属性的set方法,由于之前执行render的时候触发了get,收集了对应的Watcher,所以改变值时触发set,通知之前收集的Watcher实例执行,重新计算render方法进行patch操作

最后盗取一张图:

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/4/1632ab0bb959ae7f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)



写了半天,实在写不下去了,以后有好的语言,再来整理吧!



作者：litongqian
链接：https://juejin.cn/post/6844903601953849352
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





## 前言

大家好，我是村长，一个爱分享的老前端。前一段时间有很多小伙伴找我做经典Vue面试题解析。我整整花了1个月时间找题写答案，录视频，所有内容均为原创。本来打算全部写完再来发一篇鸿篇巨制，不过找我要学习材料的小伙伴太多，只好提前发布。这里是已经写完的前半部分，大家务必`点赞`、`收藏`保存一下，我后续还在这一篇更新下半部分，希望大家多多支持村长，感谢。

## 学习群

我组织了一个面试学习群，关注村长公众号`村长学前端`，回复“加群”，大家一起卷~

## 相关学习资源

本系列有`配套视频`，`思维导图`和`开源项目`，大家学习同时千万不要忘了`三连` + `关注` + `分享`,有道是喝水不忘挖井人~

- 视频教程：[【Vue面试专题】金三银四必备！56道经典Vue面试题详](https://link.juejin.cn?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV11i4y1Q7H2%2F)
- 思维导图：[Vue面试专题](https://link.juejin.cn?target=https%3A%2F%2Fwww.processon.com%2Fview%2Flink%2F620c4de01efad406e72b891f%23map)
- 配套代码：[vue-interview](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2F57code%2Fvue-interview)

------

## 01-Vue组件之间通信方式有哪些

vue是组件化开发框架，所以对于vue应用来说组件间的数据通信非常重要。 此题主要考查大家vue基本功，对于vue基础api运用熟练度。 另外一些边界知识如provide/inject/$attrs则提现了面试者的知识广度。

------

组件传参的各种方式 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf775050e1f948bfa52f3c79b3a3e538~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

### 思路分析：

1. 总述知道的所有方式
2. 按组件关系阐述使用场景

------

### 回答范例：

1. 组件通信常用方式有以下8种：

- props
- $emit/~~$on~~
- ~~$children~~/$parent
- $attrs/~~$listeners~~
- ref
- $root
- eventbus
- vuex

> 注意vue3中废弃的几个API
>
> [v3-migration.vuejs.org/breaking-ch…](https://link.juejin.cn?target=https%3A%2F%2Fv3-migration.vuejs.org%2Fbreaking-changes%2Fchildren.html)
>
> [v3-migration.vuejs.org/breaking-ch…](https://link.juejin.cn?target=https%3A%2F%2Fv3-migration.vuejs.org%2Fbreaking-changes%2Flisteners-removed.html)
>
> [v3-migration.vuejs.org/breaking-ch…](https://link.juejin.cn?target=https%3A%2F%2Fv3-migration.vuejs.org%2Fbreaking-changes%2Fevents-api.html%23overview)

------

1. 根据组件之间关系讨论组件通信最为清晰有效

- 父子组件
  - `props`/`$emit`/`$parent`/`ref`/`$attrs`
- 兄弟组件
  - `$parent`/`$root`/`eventbus`/`vuex`
- 跨层级关系
  - `eventbus`/`vuex`/`provide`+`inject`

------

## 02-v-if和v-for哪个优先级更高？

### 分析：

此题考查常识，文档中曾有详细说明[v2](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fstyle-guide%2F%23%E9%81%BF%E5%85%8D-v-if-%E5%92%8C-v-for-%E7%94%A8%E5%9C%A8%E4%B8%80%E8%B5%B7%E5%BF%85%E8%A6%81)|[v3](https://link.juejin.cn?target=https%3A%2F%2Fstaging.vuejs.org%2Fstyle-guide%2Frules-essential.html%23avoid-v-if-with-v-for)；也是一个很好的实践题目，项目中经常会遇到，能够看出面试者api熟悉程度和应用能力。

------

### 思路分析：

1. 先给出结论
2. 为什么是这样的，说出细节
3. 哪些场景可能导致我们这样做，该怎么处理
4. 总结，拔高

------

### 回答范例：

1. 实践中**不应该把v-for和v-if放一起**
2. 在**vue2中**，**v-for的优先级是高于v-if**，把它们放在一起，输出的渲染函数中可以看出会先执行循环再判断条件，哪怕我们只渲染列表中一小部分元素，也得在每次重渲染的时候遍历整个列表，这会比较浪费；另外需要注意的是在**vue3中则完全相反，v-if的优先级高于v-for**，所以v-if执行时，它调用的变量还不存在，就会导致异常
3. 通常有两种情况下导致我们这样做：
   - 为了**过滤列表中的项目** (比如 `v-for="user in users" v-if="user.isActive"`)。此时定义一个计算属性 (比如 `activeUsers`)，让其返回过滤后的列表即可（比如`users.filter(u=>u.isActive)`）。
   - 为了**避免渲染本应该被隐藏的列表** (比如 `v-for="user in users" v-if="shouldShowUsers"`)。此时把 `v-if` 移动至容器元素上 (比如 `ul`、`ol`)或者外面包一层`template`即可。
4. 文档中明确指出**永远不要把 `v-if` 和 `v-for` 同时用在同一个元素上**，显然这是一个重要的注意事项。
5. 源码里面关于代码生成的部分，能够清晰的看到是先处理v-if还是v-for，顺序上vue2和vue3正好相反，因此产生了一些症状的不同，但是不管怎样都是不能把它们写在一起的。

------

### 知其所以然：

做个测试，[test.html](https://link.juejin.cn?target=.%2Ftest.html) 两者同级时，渲染函数如下：

```js
ƒ anonymous(
) {
with(this){return _c('div',{attrs:{"id":"app"}},_l((items),function(item){return (item.isActive)?_c('div',{key:item.id},[_v("\n      "+_s(item.name)+"\n    ")]):_e()}),0)}
}
复制代码
```

------

做个测试，test-v3.html

![image-20220210104854185](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c265563dcbf4dbab2b889ac72d8f654~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

源码中找答案

v2：[github1s.com/vuejs/vue/b…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fvue%2Fblob%2FHEAD%2Fsrc%2Fcompiler%2Fcodegen%2Findex.js%23L65-L66)

v3：[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fcompiler-core%2Fsrc%2Fcodegen.ts%23L586-L587)

------

## 03-简述 Vue 的生命周期以及每个阶段做的事

必问题目，考查vue基础知识。

### 思路

1. 给出概念
2. 列举生命周期各阶段
3. 阐述整体流程
4. 结合实践
5. 扩展：vue3变化

------

### 回答范例

1.每个Vue组件实例被创建后都会经过一系列初始化步骤，比如，它需要数据观测，模板编译，挂载实例到dom上，以及数据变化时更新dom。这个过程中会运行叫做生命周期钩子的函数，以便用户在特定阶段有机会添加他们自己的代码。

2.Vue生命周期总共可以分为8个阶段：**创建前后, 载入前后, 更新前后, 销毁前后**，以及一些特殊场景的生命周期。vue3中新增了三个用于调试和服务端渲染场景。

------

| 生命周期v2    | 生命周期v3        | 描述                       |
| ------------- | ----------------- | -------------------------- |
| beforeCreate  | beforeCreate      | 组件实例被创建之初         |
| created       | created           | 组件实例已经完全创建       |
| beforeMount   | beforeMount       | 组件挂载之前               |
| mounted       | mounted           | 组件挂载到实例上去之后     |
| beforeUpdate  | beforeUpdate      | 组件数据发生变化，更新之前 |
| updated       | updated           | 数据数据更新之后           |
| beforeDestroy | **beforeUnmount** | 组件实例销毁之前           |
| destroyed     | **unmounted**     | 组件实例销毁之后           |

------

| 生命周期v2    | 生命周期v3          | 描述                                     |
| ------------- | ------------------- | ---------------------------------------- |
| activated     | activated           | keep-alive 缓存的组件激活时              |
| deactivated   | deactivated         | keep-alive 缓存的组件停用时调用          |
| errorCaptured | errorCaptured       | 捕获一个来自子孙组件的错误时被调用       |
| -             | **renderTracked**   | 调试钩子，响应式依赖被收集时调用         |
| -             | **renderTriggered** | 调试钩子，响应式依赖被触发时调用         |
| -             | **serverPrefetch**  | ssr only，组件实例在服务器上被渲染前调用 |

------

3.[`Vue`生命周期流程图](https://link.juejin.cn?target=https%3A%2F%2Fgitee.com%2F57code%2Fpicgo%2Fraw%2Fmaster%2Flifecycle.cec11dcc.png)：

![Component lifecycle diagram](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/779f7121823d4118a5b6ad2aa4007c28~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

4.结合实践：

**beforeCreate**：通常用于插件开发中执行一些初始化任务

**created**：组件初始化完毕，可以访问各种数据，获取接口数据等

**mounted**：dom已创建，可用于获取访问数据和dom元素；访问子组件等。

**beforeUpdate**：此时`view`层还未更新，可用于获取更新前各种状态

**updated**：完成`view`层的更新，更新后，所有状态已是最新

**beforeunmount**：实例被销毁前调用，可用于一些定时器或订阅的取消

**unmounted**：销毁一个实例。可清理它与其它实例的连接，解绑它的全部指令及事件监听器

### 可能的追问

1. setup和created谁先执行？
2. setup中为什么没有beforeCreate和created？

------

### 知其所以然

vue3中生命周期的派发时刻：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FcomponentOptions.ts%23L554-L555)

vue2中声明周期的派发时刻：

[github1s.com/vuejs/vue/b…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fvue%2Fblob%2FHEAD%2Fsrc%2Fcore%2Finstance%2Finit.js%23L55-L56)

------

## 04-能说一说双向绑定使用和原理吗？

### 题目分析：

双向绑定是`vue`的特色之一，开发中必然会用到的知识点，然而此题还问了实现原理，升级为深度考查。

------

### 思路分析：

1. 给出双绑定义
2. 双绑带来的好处
3. 在哪使用双绑
4. 使用方式、使用细节、vue3变化
5. 原理实现描述

------

### 回答范例：

1. vue中双向绑定是一个指令`v-model`，可以绑定一个响应式数据到视图，同时视图中变化能改变该值。
2. `v-model`是语法糖，默认情况下相当于`:value`和`@input`。使用`v-model`可以减少大量繁琐的事件处理代码，提高开发效率。
3. 通常在表单项上使用`v-model`，还可以在自定义组件上使用，表示某个值的输入和输出控制。
4. 通过``的方式将xxx的值绑定到表单元素value上；对于checkbox，可以使用`true-value`和false-value指定特殊的值，对于radio可以使用value指定特殊的值；对于select可以通过options元素的value设置特殊的值；还可以结合.lazy,.number,.trim对v-mode的行为做进一步限定；`v-model`用在自定义组件上时又会有很大不同，vue3中它类似于`sync`修饰符，最终展开的结果是modelValue属性和update:modelValue事件；vue3中我们甚至可以用参数形式指定多个不同的绑定，例如v-model:foo和v-model:bar，非常强大！
5. `v-model`是一个指令，它的神奇魔法实际上是vue的编译器完成的。我做过测试，包含`v-model`的模板，转换为渲染函数之后，实际上还是是value属性的绑定以及input事件监听，事件回调函数中会做相应变量更新操作。编译器根据表单元素的不同会展开不同的DOM属性和事件对，比如text类型的input和textarea会展开为value和input事件；checkbox和radio类型的input会展开为checked和change事件；select用value作为属性，用change作为事件。

------

### 可能的追问：

1. `v-model`和`sync`修饰符有什么区别
2. 自定义组件使用`v-model`如果想要改变事件名或者属性名应该怎么做

------

### 知其所以然：

测试代码，[test.html](https://link.juejin.cn?target=.%2Ftest.html)

观察输出的渲染函数：

```js
// <input type="text" v-model="foo">
_c('input', { 
  directives: [{ name: "model", rawName: "v-model", value: (foo), expression: "foo" }], 
  attrs: { "type": "text" }, 
  domProps: { "value": (foo) }, 
  on: { 
    "input": function ($event) { 
      if ($event.target.composing) return; 
      foo = $event.target.value 
    } 
  } 
})
复制代码
```

------

```js
// <input type="checkbox" v-model="bar">
_c('input', { 
  directives: [{ name: "model", rawName: "v-model", value: (bar), expression: "bar" }], 
  attrs: { "type": "checkbox" }, 
  domProps: { 
    "checked": Array.isArray(bar) ? _i(bar, null) > -1 : (bar) 
  }, 
  on: { 
    "change": function ($event) { 
      var $$a = bar, $$el = $event.target, $$c = $$el.checked ? (true) : (false); 
      if (Array.isArray($$a)) { 
        var $$v = null, $$i = _i($$a, $$v); 
        if ($$el.checked) { $$i < 0 && (bar = $$a.concat([$$v])) } 
        else { 
          $$i > -1 && (bar = $$a.slice(0, $$i).concat($$a.slice($$i + 1))) } 
      } else { 
        bar = $$c 
      } 
    } 
  } 
})
复制代码
```

------

```js
// <select v-model="baz">
//     <option value="vue">vue</option>
//     <option value="react">react</option>
// </select>
_c('select', { 
  directives: [{ name: "model", rawName: "v-model", value: (baz), expression: "baz" }], 
  on: { 
    "change": function ($event) { 
      var $$selectedVal = Array.prototype.filter.call(
        $event.target.options, 
        function (o) { return o.selected }
      ).map(
        function (o) { 
          var val = "_value" in o ? o._value : o.value; 
          return val 
        }
      ); 
      baz = $event.target.multiple ? $$selectedVal : $$selectedVal[0] 
    } 
  } 
}, [
  _c('option', { attrs: { "value": "vue" } }, [_v("vue")]), _v(" "), 
  _c('option', { attrs: { "value": "react" } }, [_v("react")])
])
复制代码
```

------

## 05-Vue中如何扩展一个组件

此题属于实践题，考察大家对vue常用api使用熟练度，答题时不仅要列出这些解决方案，同时最好说出他们异同。

### 答题思路：

1. 按照逻辑扩展和内容扩展来列举，
   - 逻辑扩展有：mixins、extends、composition api；
   - 内容扩展有slots；
2. 分别说出他们使用方法、场景差异和问题。
3. 作为扩展，还可以说说vue3中新引入的composition api带来的变化

------

### 回答范例：

1. 常见的组件扩展方法有：mixins，slots，extends等

2. 混入mixins是分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。

   ```js
   // 复用代码：它是一个配置对象，选项和组件里面一样
   const mymixin = {
      methods: {
         dosomething(){}
      }
   }
   // 全局混入：将混入对象传入
   Vue.mixin(mymixin)
   
   // 局部混入：做数组项设置到mixins选项，仅作用于当前组件
   const Comp = {
      mixins: [mymixin]
   }
   复制代码
   ```

------

1. 插槽主要用于vue组件中的内容分发，也可以用于组件扩展。

   子组件Child

   ```html
   <div>
     <slot>这个内容会被父组件传递的内容替换</slot>
   </div>
   复制代码
   ```

   父组件Parent

   ```html
   <div>
      <Child>来自老爹的内容</Child>
   </div>
   复制代码
   ```

   如果要精确分发到不同位置可以使用具名插槽，如果要使用子组件中的数据可以使用作用域插槽。

------

1. 组件选项中还有一个不太常用的选项extends，也可以起到扩展组件的目的

   ```js
   // 扩展对象
   const myextends = {
      methods: {
         dosomething(){}
      }
   }
   // 组件扩展：做数组项设置到extends选项，仅作用于当前组件
   // 跟混入的不同是它只能扩展单个对象
   // 另外如果和混入发生冲突，该选项优先级较高，优先起作用
   const Comp = {
      extends: myextends
   }
   复制代码
   ```

------

1. 混入的数据和方法**不能明确判断来源**且可能和当前组件内变量**产生命名冲突**，vue3中引入的composition api，可以很好解决这些问题，利用独立出来的响应式模块可以很方便的编写独立逻辑并提供响应式的数据，然后在setup选项中组合使用，增强代码的可读性和维护性。例如：

   ```js
   // 复用逻辑1
   function useXX() {}
   // 复用逻辑2
   function useYY() {}
   // 逻辑组合
   const Comp = {
      setup() {
         const {xx} = useXX()
         const {yy} = useYY()
         return {xx, yy}
      }
   }
   复制代码
   ```

------

### 可能的追问

Vue.extend方法你用过吗？它能用来做组件扩展吗？

------

### 知其所以然

mixins原理：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FapiCreateApp.ts%23L232-L233)

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FcomponentOptions.ts%23L545)

slots原理：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FcomponentSlots.ts%23L129-L130)

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L1373-L1374)

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fhelpers%2FrenderSlot.ts%23L23-L24)

------

## 06-子组件可以直接改变父组件的数据么，说明原因

### 分析

这是一个实践知识点，组件化开发过程中有个**单项数据流原则**，不在子组件中修改父组件是个常识问题。

参考文档：[staging.vuejs.org/guide/compo…](https://link.juejin.cn?target=https%3A%2F%2Fstaging.vuejs.org%2Fguide%2Fcomponents%2Fprops.html%23one-way-data-flow)

------

### 思路

1. 讲讲单项数据流原则，表明为何不能这么做
2. 举几个常见场景的例子说说解决方案
3. 结合实践讲讲如果需要修改父组件状态应该如何做

------

### 回答范例

1. 所有的 prop 都使得其父子之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外变更父级组件的状态，从而导致你的应用的数据流向难以理解。另外，每次父级组件发生变更时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你**不**应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器控制台中发出警告。

   ```ini
   const props = defineProps(['foo'])
   // ❌ 下面行为会被警告, props是只读的!
   props.foo = 'bar'
   复制代码
   ```

------

1. 实际开发过程中有两个场景会想要修改一个属性：

   - **这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用。**在这种情况下，最好定义一个本地的 data，并将这个 prop 用作其初始值：

     ```js
     const props = defineProps(['initialCounter'])
     const counter = ref(props.initialCounter)
     复制代码
     ```

   - **这个 prop 以一种原始的值传入且需要进行转换。**在这种情况下，最好使用这个 prop 的值来定义一个计算属性：

     ```js
     const props = defineProps(['size'])
     // prop变化，计算属性自动更新
     const normalizedSize = computed(() => props.size.trim().toLowerCase())
     复制代码
     ```

2. 实践中如果确实想要改变父组件属性应该emit一个事件让父组件去做这个变更。注意虽然我们不能直接修改一个传入的对象或者数组类型的prop，但是我们还是能够直接改内嵌的对象或属性。

------

## 07-Vue要做权限管理该怎么做？控制到按钮级别的权限怎么做？

### 分析

综合实践题目，实际开发中经常需要面临权限管理的需求，考查实际应用能力。

权限管理一般需求是两个：页面权限和按钮权限，从这两个方面论述即可。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/631e5a9510f349e488227498ec6212e9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

### 思路

1. 权限管理需求分析：页面和按钮权限
2. 权限管理的实现方案：分后端方案和前端方案阐述
3. 说说各自的优缺点

------

### 回答范例

1. 权限管理一般需求是**页面权限**和**按钮权限**的管理

2. 具体实现的时候分后端和前端两种方案：

   前端方案会**把所有路由信息在前端配置**，通过路由守卫要求用户登录，用户**登录后根据角色过滤出路由表**。比如我会配置一个`asyncRoutes`数组，需要认证的页面在其路由的`meta`中添加一个`roles`字段，等获取用户角色之后取两者的交集，若结果不为空则说明可以访问。此过滤过程结束，剩下的路由就是该用户能访问的页面，**最后通过`router.addRoutes(accessRoutes)`方式动态添加路由**即可。

   后端方案会**把所有页面路由信息存在数据库**中，用户登录的时候根据其角色**查询得到其能访问的所有页面路由信息**返回给前端，前端**再通过`addRoutes`动态添加路由**信息

   按钮权限的控制通常会**实现一个指令**，例如`v-permission`，**将按钮要求角色通过值传给v-permission指令**，在指令的`moutned`钩子中可以**判断当前用户角色和按钮是否存在交集**，有则保留按钮，无则移除按钮。

3. 纯前端方案的优点是实现简单，不需要额外权限管理页面，但是维护起来问题比较大，有新的页面和角色需求就要修改前端代码重新打包部署；服务端方案就不存在这个问题，通过专门的角色和权限管理页面，配置页面和按钮权限信息到数据库，应用每次登陆时获取的都是最新的路由信息，可谓一劳永逸！

------

### 知其所以然

路由守卫

[github1s.com/PanJiaChen/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2FPanJiaChen%2Fvue-element-admin%2Fblob%2FHEAD%2Fsrc%2Fpermission.js%23L13-L14)

路由生成

[github1s.com/PanJiaChen/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2FPanJiaChen%2Fvue-element-admin%2Fblob%2FHEAD%2Fsrc%2Fstore%2Fmodules%2Fpermission.js%23L50-L51)

动态追加路由

[github1s.com/PanJiaChen/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2FPanJiaChen%2Fvue-element-admin%2Fblob%2FHEAD%2Fsrc%2Fpermission.js%23L43-L44)

------

### 可能的追问

1. 类似`Tabs`这类组件能不能使用`v-permission`指令实现按钮权限控制？

   ```html
   <el-tabs> 
     <el-tab-pane label="⽤户管理" name="first">⽤户管理</el-tab-pane> 
   	<el-tab-pane label="⻆⾊管理" name="third">⻆⾊管理</el-tab-pane>
   </el-tabs>
   复制代码
   ```

------

1. 服务端返回的路由信息如何添加到路由器中？

   ```js
   // 前端组件名和组件映射表
   const map = {
     //xx: require('@/views/xx.vue').default // 同步的⽅式
     xx: () => import('@/views/xx.vue') // 异步的⽅式
   }
   // 服务端返回的asyncRoutes
   const asyncRoutes = [
     { path: '/xx', component: 'xx',... }
   ]
   // 遍历asyncRoutes，将component替换为map[component]
   function mapComponent(asyncRoutes) {
     asyncRoutes.forEach(route => {
       route.component = map[route.component];
       if(route.children) {
         route.children.map(child => mapComponent(child))
       }
   	})
   }
   mapComponent(asyncRoutes)
   复制代码
   ```

------

## 08 - 说一说你对vue响应式理解？

### 分析

这是一道必问题目，但能回答到位的比较少。如果只是看看一些网文，通常没什么底气，经不住面试官推敲，但像我们这样即看过源码还造过轮子的，回答这个问题就会比较有底气啦。

### 答题思路：

1. 啥是响应式？
2. 为什么vue需要响应式？
3. 它能给我们带来什么好处？
4. vue的响应式是怎么实现的？有哪些优缺点？
5. vue3中的响应式的新变化

------

### 回答范例：

1. 所谓数据响应式就是**能够使数据变化可以被检测并对这种变化做出响应的机制**。
2. MVVM框架中要解决的一个核心问题是连接数据层和视图层，通过**数据驱动**应用，数据变化，视图更新，要做到这点的就需要对数据做响应式处理，这样一旦数据发生变化就可以立即做出更新处理。
3. 以vue为例说明，通过数据响应式加上虚拟DOM和patch算法，开发人员只需要操作数据，关心业务，完全不用接触繁琐的DOM操作，从而大大提升开发效率，降低开发难度。
4. vue2中的数据响应式会根据数据类型来做不同处理，如果是**对象则采用Object.defineProperty()\**的方式定义数据拦截，当数据被访问或发生变化时，我们感知并作出响应；如果是\**数组则通过覆盖数组对象原型的7个变更方法**，使这些方法可以额外的做更新通知，从而作出响应。这种机制很好的解决了数据响应化的问题，但在实际使用中也存在一些缺点：比如初始化时的递归遍历会造成性能损失；新增或删除属性时需要用户使用Vue.set/delete这样特殊的api才能生效；对于es6中新产生的Map、Set这些数据结构不支持等问题。
5. 为了解决这些问题，vue3重新编写了这一部分的实现：利用ES6的Proxy代理要响应化的数据，它有很多好处，编程体验是一致的，不需要使用特殊api，初始化性能和内存消耗都得到了大幅改善；另外由于响应化的实现代码抽取为独立的reactivity包，使得我们可以更灵活的使用它，第三方的扩展开发起来更加灵活了。

------

### 知其所以然

vue2响应式：

[github1s.com/vuejs/vue/b…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fvue%2Fblob%2FHEAD%2Fsrc%2Fcore%2Fobserver%2Findex.js%23L135-L136)

vue3响应式：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Freactive.ts%23L89-L90)

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Fref.ts%23L67-L68)

------

## 09 - 说说你对虚拟 DOM 的理解？

### 分析

现有框架几乎都引入了虚拟 DOM 来对真实 DOM 进行抽象，也就是现在大家所熟知的 VNode 和 VDOM，那么为什么需要引入虚拟 DOM 呢？围绕这个疑问来解答即可！

### 思路

1. vdom是什么
2. 引入vdom的好处
3. vdom如何生成，又如何成为dom
4. 在后续的diff中的作用

------

### 回答范例

1. 虚拟dom顾名思义就是虚拟的dom对象，它本身就是一个 `JavaScript` 对象，只不过它是通过不同的属性去描述一个视图结构。

2. 通过引入vdom我们可以获得如下好处：

   **将真实元素节点抽象成 VNode，有效减少直接操作 dom 次数，从而提高程序性能**

   - 直接操作 dom 是有限制的，比如：diff、clone 等操作，一个真实元素上有许多的内容，如果直接对其进行 diff 操作，会去额外 diff 一些没有必要的内容；同样的，如果需要进行 clone 那么需要将其全部内容进行复制，这也是没必要的。但是，如果将这些操作转移到 JavaScript 对象上，那么就会变得简单了。
   - 操作 dom 是比较昂贵的操作，频繁的dom操作容易引起页面的重绘和回流，但是通过抽象 VNode 进行中间处理，可以有效减少直接操作dom的次数，从而减少页面重绘和回流。

   **方便实现跨平台**

   - 同一 VNode 节点可以渲染成不同平台上的对应的内容，比如：渲染在浏览器是 dom 元素节点，渲染在 Native( iOS、Android) 变为对应的控件、可以实现 SSR 、渲染到 WebGL 中等等
   - Vue3 中允许开发者基于 VNode 实现自定义渲染器（renderer），以便于针对不同平台进行渲染。

------

1. vdom如何生成？在vue中我们常常会为组件编写模板 - template， 这个模板会被编译器 - compiler编译为渲染函数，在接下来的挂载（mount）过程中会调用render函数，返回的对象就是虚拟dom。但它们还不是真正的dom，所以会在后续的patch过程中进一步转化为dom。

   ![image-20220209153820845](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80b653050433436da876459a26ab5a65~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

2. 挂载过程结束后，vue程序进入更新流程。如果某些响应式数据发生变化，将会引起组件重新render，此时就会生成新的vdom，和上一次的渲染结果diff就能得到变化的地方，从而转换为最小量的dom操作，高效更新视图。

------

### 知其所以然

vnode定义：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fvnode.ts%23L127-L128)

观察渲染函数：21-vdom/test-render-v3.html

创建vnode：

- createElementBlock:

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fvnode.ts%23L291-L292)

- createVnode:

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fvnode.ts%23L486-L487)

- 首次调用时刻：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FapiCreateApp.ts%23L283-L284)

------

mount:

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L1171-L1172)

调试mount过程：mountComponent

21-vdom/test-render-v3.html

------

## 10 - 你了解diff算法吗？

### 分析

必问题目，涉及vue更新原理，比较考查理解深度。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bffb8ffca9f0468c8a31576cebe6e692~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

### 思路

1. diff算法是干什么的
2. 它的必要性
3. 它何时执行
4. 具体执行方式
5. 拔高：说一下vue3中的优化

------

### 回答范例

1.Vue中的diff算法称为patching算法，它由Snabbdom修改而来，虚拟DOM要想转化为真实DOM就需要通过patch方法转换。

2.最初Vue1.x视图中每个依赖均有更新函数对应，可以做到精准更新，因此并不需要虚拟DOM和patching算法支持，但是这样粒度过细导致Vue1.x无法承载较大应用；Vue 2.x中为了降低Watcher粒度，每个组件只有一个Watcher与之对应，此时就需要引入patching算法才能精确找到发生变化的地方并高效更新。

3.vue中diff执行的时刻是组件内响应式数据变更触发实例执行其更新函数时，更新函数会再次执行render函数获得最新的虚拟DOM，然后执行patch函数，并传入新旧两次虚拟DOM，通过比对两者找到变化的地方，最后将其转化为对应的DOM操作。

------

4.patch过程是一个递归过程，遵循深度优先、同层比较的策略；以vue3的patch为例：

- 首先判断两个节点是否为相同同类节点，不同则删除重新创建
- 如果双方都是文本则更新文本内容
- 如果双方都是元素节点则递归更新子元素，同时更新元素属性
- 更新子节点时又分了几种情况： 
  - 新的子节点是文本，老的子节点是数组则清空，并设置文本；
  - 新的子节点是文本，老的子节点是文本则直接更新文本；
  - 新的子节点是数组，老的子节点是文本则清空文本，并创建新子节点数组中的子元素；
  - 新的子节点是数组，老的子节点也是数组，那么比较两组子节点，更新细节blabla

1. vue3中引入的更新策略：编译期优化patchFlags、block等

------

### 知其所以然

patch关键代码

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L354-L355)

调试 [test-v3.html](https://link.juejin.cn?target=text-v3.html)

------

## 11 - 你知道哪些vue3新特性

### 分析

官网列举的最值得注意的新特性：[v3-migration.vuejs.org/](https://link.juejin.cn?target=https%3A%2F%2Fv3-migration.vuejs.org%2F)

![image-20220210165307624](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e52235d31934130914925042b96e3a7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

也就是下面这些：

- Composition API
- SFC Composition API语法糖
- Teleport传送门
- Fragments片段
- Emits选项
- 自定义渲染器
- SFC CSS变量
- Suspense

以上这些是api相关，另外还有很多框架特性也不能落掉。

------

### 回答范例

1. api层面Vue3新特性主要包括：Composition API、SFC Composition API语法糖、Teleport传送门、Fragments 片段、Emits选项、自定义渲染器、SFC CSS变量、Suspense
2. 另外，Vue3.0在框架层面也有很多亮眼的改进：

- 更快 
  - 虚拟DOM重写
  - 编译器优化：静态提升、patchFlags、block等
  - 基于Proxy的响应式系统
- 更小：更好的摇树优化
- 更容易维护：TypeScript + 模块化
- 更容易扩展 
  - 独立的响应化模块
  - 自定义渲染器

------

### 知其所以然

体验编译器优化

[sfc.vuejs.org/](https://link.juejin.cn?target=https%3A%2F%2Fsfc.vuejs.org%2F)

reactive实现

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Freactive.ts%23L90-L91)

------

## 12 - 怎么定义动态路由？怎么获取传过来的动态参数？

### 分析

API题目，考查基础能力，不容有失，尽可能说的详细。

### 思路

1. 什么是动态路由
2. 什么时候使用动态路由，怎么定义动态路由
3. 参数如何获取
4. 细节、注意事项

------

### 回答范例

1. 很多时候，我们需要**将给定匹配模式的路由映射到同一个组件**，这种情况就需要定义动态路由。
2. 例如，我们可能有一个 `User` 组件，它应该对所有用户进行渲染，但用户 ID 不同。在 Vue Router 中，我们可以在路径中使用一个动态字段来实现，例如：`{ path: '/users/:id', component: User }`，其中`:id`就是路径参数
3. *路径参数* 用冒号 `:` 表示。当一个路由被匹配时，它的 *params* 的值将在每个组件中以 `this.$route.params` 的形式暴露出来。
4. 参数还可以有多个，例如`/users/:username/posts/:postId`；除了 `$route.params` 之外，`$route` 对象还公开了其他有用的信息，如 `$route.query`、`$route.hash` 等。

------

### 可能的追问

1. 如何响应动态路由参数的变化

[router.vuejs.org/zh/guide/es…](https://link.juejin.cn?target=https%3A%2F%2Frouter.vuejs.org%2Fzh%2Fguide%2Fessentials%2Fdynamic-matching.html%23%E5%93%8D%E5%BA%94%E8%B7%AF%E7%94%B1%E5%8F%82%E6%95%B0%E7%9A%84%E5%8F%98%E5%8C%96)

1. 我们如何处理404 Not Found路由

[router.vuejs.org/zh/guide/es…](https://link.juejin.cn?target=https%3A%2F%2Frouter.vuejs.org%2Fzh%2Fguide%2Fessentials%2Fdynamic-matching.html%23%E6%8D%95%E8%8E%B7%E6%89%80%E6%9C%89%E8%B7%AF%E7%94%B1%E6%88%96-404-not-found-%E8%B7%AF%E7%94%B1)

------

## 13-如果让你从零开始写一个vue路由，说说你的思路

### 思路分析：

首先思考vue路由要解决的问题：用户点击跳转链接内容切换，页面不刷新。

- 借助hash或者history api实现url跳转页面不刷新
- 同时监听hashchange事件或者popstate事件处理跳转
- 根据hash值或者state值从routes表中匹配对应component并渲染之

------

### 回答范例：

一个SPA应用的路由需要解决的问题是**页面跳转内容改变同时不刷新**，同时路由还需要以插件形式存在，所以：

1. 首先我会定义一个

   ```
   createRouter
   ```

   函数，返回路由器实例，实例内部做几件事： 

   - 保存用户传入的配置项
   - 监听hash或者popstate事件
   - 回调里根据path匹配对应路由

2. 将router定义成一个Vue插件，即实现install方法，内部做两件事： 

   - 实现两个全局组件：router-link和router-view，分别实现页面跳转和内容显示
   - 定义两个全局变量：$route和$router，组件内可以访问当前路由和路由器实例

------

### 知其所以然：

- createRouter如何创建实例

[github1s.com/vuejs/route…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Frouter%2Fblob%2FHEAD%2Fsrc%2Frouter.ts%23L355-L356)

- 事件监听

[github1s.com/vuejs/route…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Frouter%2Fblob%2FHEAD%2Fsrc%2Fhistory%2Fhtml5.ts%23L314-L315) RouterView

- 页面跳转RouterLink

[github1s.com/vuejs/route…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Frouter%2Fblob%2FHEAD%2Fsrc%2FRouterLink.ts%23L184-L185)

- 内容显示RouterView

[github1s.com/vuejs/route…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Frouter%2Fblob%2FHEAD%2Fsrc%2FRouterView.ts%23L43-L44)

------

## 14-能说说key的作用吗？

### 分析：

这是一道特别常见的问题，主要考查大家对虚拟DOM和patch细节的掌握程度，能够反映面试者理解层次。

------

### 思路分析：

1. 给出结论，key的作用是用于优化patch性能
2. key的必要性
3. 实际使用方式
4. 总结：可从源码层面描述一下vue如何判断两个节点是否相同

------

### 回答范例：

1. key的作用主要是为了更高效的更新虚拟DOM。
2. vue在patch过程中**判断两个节点是否是相同节点是key是一个必要条件**，渲染一组列表时，key往往是唯一标识，所以如果不定义key的话，vue只能认为比较的两个节点是同一个，哪怕它们实际上不是，这导致了频繁更新元素，使得整个patch过程比较低效，影响性能。
3. 实际使用中在渲染一组列表时key必须设置，而且必须是唯一标识，应该避免使用数组索引作为key，这可能导致一些隐蔽的bug；vue中在使用相同标签元素过渡切换时，也会使用key属性，其目的也是为了让vue可以区分它们，否则vue只会替换其内部属性而不会触发过渡效果。
4. 从源码中可以知道，vue判断两个节点是否相同时主要判断两者的key和元素类型等，因此如果不设置key，它的值就是undefined，则可能永远认为这是两个相同节点，只能去做更新操作，这造成了大量的dom更新操作，明显是不可取的。

------

### 知其所以然

测试代码，[test-v3.html](https://link.juejin.cn?target=.%2Ftest-v3.html)

上面案例重现的是以下过程

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d6750831c024116bcac3ece9497b597~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

不使用key

![image-20220214110059028](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d393bc0c4a8946a9a20973b64633852f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

如果使用key

```js
// 首次循环patch A
A B C D E
A B F C D E

// 第2次循环patch B
B C D E
B F C D E

// 第3次循环patch E
C D E
F C D E

// 第4次循环patch D
C D
F C D

// 第5次循环patch C
C 
F C

// oldCh全部处理结束，newCh中剩下的F，创建F并插入到C前面
复制代码
```

------

源码中找答案：

判断是否为相同节点

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fvnode.ts%23L342-L343)

更新时的处理

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L1752-L1753)

------

不使用key

![image-20220214110059028](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39eb54d8c667473ca344382782f7042a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

如果使用key

```js
// 首次循环patch A
A B C D E
A B F C D E

// 第2次循环patch B
B C D E
B F C D E

// 第3次循环patch E
C D E
F C D E

// 第4次循环patch D
C D
F C D

// 第5次循环patch C
C 
F C

// oldCh全部处理结束，newCh中剩下的F，创建F并插入到C前面
复制代码
```

源码中找答案：

判断是否为相同节点

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fvnode.ts%23L342-L343)

更新时的处理

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L1752-L1753)

------

## 15-说说nextTick的使用和原理？

### 分析

这道题及考察使用，有考察原理，nextTick在开发过程中应用的也较少，原理上和vue异步更新有密切关系，对于面试者考查很有区分度，如果能够很好回答此题，对面试效果有极大帮助。

### 答题思路

1. nextTick是做什么的？
2. 为什么需要它呢？
3. 开发时何时使用它？抓抓头，想想你在平时开发中使用它的地方
4. 下面介绍一下如何使用nextTick
5. 原理解读，结合异步更新和nextTick生效方式，会显得你格外优秀

------

### 回答范例：

1. [nextTick](https://link.juejin.cn?target=https%3A%2F%2Fstaging-cn.vuejs.org%2Fapi%2Fgeneral.html%23nexttick)是等待下一次 DOM 更新刷新的工具方法。
2. Vue有个异步更新策略，意思是如果数据变化，Vue不会立刻更新DOM，而是开启一个队列，把组件更新函数保存在队列中，在同一事件循环中发生的所有数据变更会异步的批量更新。这一策略导致我们对数据的修改不会立刻体现在DOM上，此时如果想要获取更新后的DOM状态，就需要使用nextTick。
3. 开发时，有两个场景我们会用到nextTick：

- created中想要获取DOM时；
- 响应式数据变化后获取DOM更新后的状态，比如希望获取列表更新后的高度。

1. nextTick签名如下：`function nextTick(callback?: () => void): Promise`

   所以我们只需要在传入的回调函数中访问最新DOM状态即可，或者我们可以await nextTick()方法返回的Promise之后做这件事。

2. 在Vue内部，nextTick之所以能够让我们看到DOM更新后的结果，是因为我们传入的callback会被添加到队列刷新函数(flushSchedulerQueue)的后面，这样等队列内部的更新函数都执行完毕，所有DOM操作也就结束了，callback自然能够获取到最新的DOM值。

------

### 知其所以然：

1. 源码解读:

组件更新函数入队：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L1547-L1548)

入队函数：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fscheduler.ts%23L84-L85)

nextTick定义：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fscheduler.ts%23L58-L59)

1. 测试案例，test-v3.html

------

## 16-watch和computed的区别以及选择?

两个重要API，反应应聘者熟练程度。

### 思路分析

1. 先看[computed](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fapi%2Freactivity-core.html%23computed), [watch](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fapi%2Freactivity-core.html%23watch)两者定义，列举使用上的差异
2. 列举使用场景上的差异，如何选择
3. 使用细节、注意事项
4. vue3变化

------

computed特点：具有响应式的返回值

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)
复制代码
```

watch特点：侦测变化，执行回调

```js
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)
复制代码
```

------

### 回答范例

1. 计算属性可以**从组件数据派生出新数据**，最常见的使用方式是设置一个函数，返回计算之后的结果，computed和methods的差异是它具备缓存性，如果依赖项不变时不会重新计算。侦听器**可以侦测某个响应式数据的变化并执行副作用**，常见用法是传递一个函数，执行副作用，watch没有返回值，但可以执行异步操作等复杂逻辑。
2. 计算属性常用场景是简化行内模板中的复杂表达式，模板中出现太多逻辑会是模板变得臃肿不易维护。侦听器常用场景是状态变化之后做一些额外的DOM操作或者异步操作。选择采用何用方案时首先看是否需要派生出新值，基本能用计算属性实现的方式首选计算属性。
3. 使用过程中有一些细节，比如计算属性也是可以传递对象，成为既可读又可写的计算属性。watch可以传递对象，设置deep、immediate等选项。
4. vue3中watch选项发生了一些变化，例如不再能侦测一个点操作符之外的字符串形式的表达式； reactivity API中新出现了watch、watchEffect可以完全替代目前的watch选项，且功能更加强大。

------

### 回答范例

1. 计算属性可以**从组件数据派生出新数据**，最常见的使用方式是设置一个函数，返回计算之后的结果，computed和methods的差异是它具备缓存性，如果依赖项不变时不会重新计算。侦听器**可以侦测某个响应式数据的变化并执行副作用**，常见用法是传递一个函数，执行副作用，watch没有返回值，但可以执行异步操作等复杂逻辑。
2. 计算属性常用场景是简化行内模板中的复杂表达式，模板中出现太多逻辑会是模板变得臃肿不易维护。侦听器常用场景是状态变化之后做一些额外的DOM操作或者异步操作。选择采用何用方案时首先看是否需要派生出新值，基本能用计算属性实现的方式首选计算属性。
3. 使用过程中有一些细节，比如计算属性也是可以传递对象，成为既可读又可写的计算属性。watch可以传递对象，设置deep、immediate等选项。
4. vue3中watch选项发生了一些变化，例如不再能侦测一个点操作符之外的字符串形式的表达式； reactivity API中新出现了watch、watchEffect可以完全替代目前的watch选项，且功能更加强大。

------

### 可能追问

1. watch会不会立即执行？
2. watch 和 watchEffect有什么差异

------

### 知其所以然

computed的实现

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Fcomputed.ts%23L79-L80)

ComputedRefImpl

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Fcomputed.ts%23L26-L27)

缓存性

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Fcomputed.ts%23L59-L60)

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Freactivity%2Fsrc%2Fcomputed.ts%23L45-L46)

watch的实现

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FapiWatch.ts%23L158-L159)

------

## 17-说一下 Vue 子组件和父组件创建和挂载顺序

这题考查大家对创建过程的理解程度。

### 思路分析

1. 给结论
2. 阐述理由

------

### 回答范例

1. 创建过程自上而下，挂载过程自下而上；即： 
   - parent created
   - child created
   - child mounted
   - parent mounted
2. 之所以会这样是因为Vue创建过程是一个递归过程，先创建父组件，有子组件就会创建子组件，因此创建时先有父组件再有子组件；子组件首次创建时会添加mounted钩子到队列，等到patch结束再执行它们，可见子组件的mounted钩子是先进入到队列中的，因此等到patch结束执行这些钩子时也先执行。

------

### 知其所以然

观察beforeCreated和created钩子的处理

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FcomponentOptions.ts%23L554-L555)

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FcomponentOptions.ts%23L741-L742)

观察beforeMount和mounted钩子的处理

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L1310-L1311)

测试代码，test-v3.html

------

## 18-怎么缓存当前的组件？缓存后怎么更新？

缓存组件使用keep-alive组件，这是一个非常常见且有用的优化手段，vue3中keep-alive有比较大的更新，能说的点比较多。

### 思路

1. 缓存用keep-alive，它的作用与用法
2. 使用细节，例如缓存指定/排除、结合router和transition
3. 组件缓存后更新可以利用activated或者beforeRouteEnter
4. 原理阐述

------

### 回答范例

1. 开发中缓存组件使用keep-alive组件，keep-alive是vue内置组件，keep-alive包裹动态组件component时，会缓存不活动的组件实例，而不是销毁它们，这样在组件切换过程中将状态保留在内存中，防止重复渲染DOM。

   ```vue
   <keep-alive>
     <component :is="view"></component>
   </keep-alive>
   复制代码
   ```

2. 结合属性include和exclude可以明确指定缓存哪些组件或排除缓存指定组件。vue3中结合vue-router时变化较大，之前是`keep-alive`包裹`router-view`，现在需要反过来用`router-view`包裹`keep-alive`：

   ```vue
   <router-view v-slot="{ Component }">
     <keep-alive>
       <component :is="Component"></component>
     </keep-alive>
   </router-view>
   复制代码
   ```

------

1. 缓存后如果要获取数据，解决方案可以有以下两种：

   - beforeRouteEnter：在有vue-router的项目，每次进入路由的时候，都会执行`beforeRouteEnter`

     ```js
     beforeRouteEnter(to, from, next){
       next(vm=>{
         console.log(vm)
         // 每次进入路由执行
         vm.getData()  // 获取数据
       })
     },
     复制代码
     ```

   - actived：在`keep-alive`缓存的组件被激活的时候，都会执行`actived`钩子

     ```js
     activated(){
     	  this.getData() // 获取数据
     },
     复制代码
     ```

------

1. keep-alive是一个通用组件，它内部定义了一个map，缓存创建过的组件实例，它返回的渲染函数内部会查找内嵌的component组件对应组件的vnode，如果该组件在map中存在就直接返回它。由于component的is属性是个响应式数据，因此只要它变化，keep-alive的render函数就会重新执行。

------

### 知其所以然

KeepAlive定义

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fcomponents%2FKeepAlive.ts%23L73-L74)

缓存定义

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fcomponents%2FKeepAlive.ts%23L102-L103)

缓存组件

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fcomponents%2FKeepAlive.ts%23L215-L216)

获取缓存组件

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Fcomponents%2FKeepAlive.ts%23L241-L242)

测试缓存特性，test-v3.html

------

## 19-从0到1自己构架一个vue项目，说说有哪些步骤、哪些重要插件、目录结构你会怎么组织

综合实践类题目，考查实战能力。没有什么绝对的正确答案，把平时工作的重点有条理的描述一下即可。

### 思路

1. 构建项目，创建项目基本结构
2. 引入必要的插件：
3. 代码规范：prettier，eslint
4. 提交规范：husky，lint-staged
5. 其他常用：svg-loader，vueuse，nprogress
6. 常见目录结构

------

### 回答范例

1. 从0创建一个项目我大致会做以下事情：项目构建、引入必要插件、代码规范、提交规范、常用库和组件
2. 目前vue3项目我会用vite或者create-vue创建项目
3. 接下来引入必要插件：路由插件vue-router、状态管理vuex/pinia、ui库我比较喜欢element-plus和antd-vue、http工具我会选axios
4. 其他比较常用的库有vueuse，nprogress，图标可以使用vite-svg-loader
5. 下面是代码规范：结合prettier和eslint即可
6. 最后是提交规范，可以使用husky，lint-staged，commitlint

------

1. 目录结构我有如下习惯： `.vscode`：用来放项目中的 vscode 配置

   `plugins`：用来放 vite 插件的 plugin 配置

   `public`：用来放一些诸如 页头icon 之类的公共文件，会被打包到dist根目录下

   `src`：用来放项目代码文件

   `api`：用来放http的一些接口配置

   `assets`：用来放一些 CSS 之类的静态资源

   `components`：用来放项目通用组件

   `layout`：用来放项目的布局

   `router`：用来放项目的路由配置

   `store`：用来放状态管理Pinia的配置

   `utils`：用来放项目中的工具方法类

   `views`：用来放项目的页面文件

------

## 20-实际工作中，你总结的vue最佳实践有哪些？

看到这样的题目，可以用以下图片来回答：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/893abf278d56465e81ac83492b150684~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

------

### 思路

查看vue官方文档：

风格指南：[vuejs.org/style-guide…](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fstyle-guide%2F)

性能：[vuejs.org/guide/best-…](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fguide%2Fbest-practices%2Fperformance.html%23overview)

安全：[vuejs.org/guide/best-…](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fguide%2Fbest-practices%2Fsecurity.html)

访问性：[vuejs.org/guide/best-…](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fguide%2Fbest-practices%2Faccessibility.html)

发布：[vuejs.org/guide/best-…](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fguide%2Fbest-practices%2Fproduction-deployment.html)

------

### 回答范例

我从编码风格、性能、安全等方面说几条：

1. 编码风格方面： 
   - 命名组件时使用“多词”风格避免和HTML元素冲突
   - 使用“细节化”方式定义属性而不是只有一个属性名
   - 属性名声明时使用“驼峰命名”，模板或jsx中使用“肉串命名”
   - 使用v-for时务必加上key，且不要跟v-if写在一起
2. 性能方面： 
   - 路由懒加载减少应用尺寸
   - 利用SSR减少首屏加载时间
   - 利用v-once渲染那些不需要更新的内容
   - 一些长列表可以利用虚拟滚动技术避免内存过度占用
   - 对于深层嵌套对象的大数组可以使用shallowRef或shallowReactive降低开销
   - 避免不必要的组件抽象

------

1. 安全： 
   - 不使用不可信模板，例如使用用户输入拼接模板：`template:  + userProvidedString + `
   - 小心使用v-html，:url，:style等，避免html、url、样式等注入
2. 等等......

------

## 21 - 简单说一说你对vuex理解？

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb128aee87e5424a83511deee98f1702~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 思路

1. 给定义
2. 必要性阐述
3. 何时使用
4. 拓展：一些个人思考、实践经验等

------

### 范例

1. Vuex 是一个专为 Vue.js 应用开发的**状态管理模式 + 库**。它采用集中式存储，管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
2. 我们期待以一种简单的“单向数据流”的方式管理应用，即状态 -> 视图 -> 操作单向循环的方式。但当我们的应用遇到**多个组件共享状态**时，比如：多个视图依赖于同一状态或者来自不同视图的行为需要变更同一状态。此时单向数据流的简洁性很容易被破坏。因此，我们有必要把组件的共享状态抽取出来，以一个全局单例模式管理。通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。这是vuex存在的必要性，它和react生态中的redux之类是一个概念。
3. Vuex 解决状态管理的同时引入了不少概念：例如state、mutation、action等，是否需要引入还需要根据应用的实际情况衡量一下：如果不打算开发大型单页应用，使用 Vuex 反而是繁琐冗余的，一个简单的 [store 模式](https://link.juejin.cn?target=https%3A%2F%2Fv3.cn.vuejs.org%2Fguide%2Fstate-management.html%23%E4%BB%8E%E9%9B%B6%E6%89%93%E9%80%A0%E7%AE%80%E5%8D%95%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86)就足够了。但是，如果要构建一个中大型单页应用，Vuex 基本是标配。
4. 我在使用vuex过程中感受到一些blabla

------

### 可能的追问

1. vuex有什么缺点吗？你在开发过程中有遇到什么问题吗？
2. action和mutation的区别是什么？为什么要区分它们？

------

## 22-说说从 template 到 render 处理过程

### 分析

问我们template到render过程，其实是问vue`编译器`工作原理。

### 思路

1. 引入vue编译器概念
2. 说明编译器的必要性
3. 阐述编译器工作流程

### 回答范例

1. Vue中有个独特的编译器模块，称为“compiler”，它的主要作用是将用户编写的template编译为js中可执行的render函数。
2. 之所以需要这个编译过程是为了便于前端程序员能高效的编写视图模板。相比而言，我们还是更愿意用HTML来编写视图，直观且高效。手写render函数不仅效率底下，而且失去了编译期的优化能力。
3. 在Vue中编译器会先对template进行解析，这一步称为parse，结束之后会得到一个JS对象，我们成为抽象语法树AST，然后是对AST进行深加工的转换过程，这一步成为transform，最后将前面得到的AST生成为JS代码，也就是render函数。

### 知其所以然

vue3编译过程窥探：

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fcompiler-core%2Fsrc%2Fcompile.ts%23L61-L62)

测试，test-v3.html

### 可能的追问

1. Vue中编译器何时执行？
2. react有没有编译器？

## 23-Vue实例挂载的过程中发生了什么?

### 分析

挂载过程完成了最重要的两件事：

1. 初始化
2. 建立更新机制

把这两件事说清楚即可！

### 回答范例

1. 挂载过程指的是app.mount()过程，这个过程中整体上做了两件事：**初始化**和**建立更新机制**
2. 初始化会创建组件实例、初始化组件状态，创建各种响应式数据
3. 建立更新机制这一步会立即执行一次组件更新函数，这会首次执行组件渲染函数并执行patch将前面获得vnode转换为dom；同时首次执行渲染函数会创建它内部响应式数据之间和组件更新函数之间的依赖关系，这使得以后数据变化时会执行对应的更新函数。

### 知其所以然

测试代码，test-v3.html mount函数定义

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2FapiCreateApp.ts%23L277-L278)

首次render过程

[github1s.com/vuejs/core/…](https://link.juejin.cn?target=https%3A%2F%2Fgithub1s.com%2Fvuejs%2Fcore%2Fblob%2FHEAD%2Fpackages%2Fruntime-core%2Fsrc%2Frenderer.ts%23L2303-L2304)

### 可能的追问

1. 响应式数据怎么创建
2. 依赖关系如何建立



作者：杨村长
链接：https://juejin.cn/post/7097067108663558151
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## Vue3新特性

### 组合式API（composition API）

**为什么使用composition API**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0f8383bd28d4e8e83d25c9db6c8a547~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

- options API的组件，比如在A组件中定义了B/C组件的data，methods，生命周期方法，computed，各个逻辑分散在组件的不同区域，**代码难以复用**，使用composition API解决了这个问题，可以做到高内聚、低耦合，**代码可复用性和可维护性**更好
- vue2逻辑复用使用的是mixins，当一个组件引用多个mixin时,想知道query来源于哪个mixin，需要在每个引用的mixin中寻找一遍方法，即**数据来源不清晰**；且多个mixin中定义的属性和方法会存在**命名冲突**问题；composition API解决了上面的问题
- 更好的类型推断，对Typescript更友好
- composition API看不到this的使用，解决了**this指向不明**的问题

```javascript
mixins:[TabMixin,TableQueryMixin,GrayBackgroundMixin,BaseQueryMixin],
methods:{
  query(){
    ...
  }
}  
复制代码
```

### teleport

Teleport类似于React的Portal，可以将组件挂载在任何DOM节点上

```xml
<button @click="openToast">打开toast</button>
<!--挂载在id为dialog的节点上-->
<teleport to="#dialog">
  <div v-if="visible" class="toast-container">
    <div class="toast-msg">我是一个toast</div>
  </div>
</teleport>
复制代码
```

### Fragment

**Fragment**组件支持多个根节点，作用：减少标签层级，减少内存占用

```xml
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
复制代码
```

### diff算法优化

### createRenderer

###  

### 生命周期变更

beforeDestory->beforeUnmount

destroyed->unmounted

更多变更详见[vue3迁移指南](https://link.juejin.cn?target=https%3A%2F%2Fwww.javascriptc.com%2Fvue3js%2Fguide%2Fmigration%2Fintroduction.html%23%E5%85%B6%E4%BB%96%E5%B0%8F%E6%94%B9%E5%8F%98)

### vue3性能提升是通过哪些方面实现的

1. 响应式系统升级，从vue2的Object.defineProperty变为了vue3的proxy，原因：

- proxy性能优于Object.defineProperty
- 可以监听数组的索引和length属性
- 可以监听动态属性的添加
- 可以监听删除属性

1. 编译优化，主要有：

- diff算法优化

vue3相比vue2增加了静态标记，静态标记的作用是会标志为一个flag，下次发生变化的时候直接找该处进行比较；已经标记为静态节点的元素不会参与diff比较

```ini
// 静态类型枚举
export const enum PatchFlags {
  TEXT = 1,// 动态的文本节点
  CLASS = 1 << 1,  // 2 动态的 class
  STYLE = 1 << 2,  // 4 动态的 style
  PROPS = 1 << 3,  // 8 动态属性，不包括类名和样式
  FULL_PROPS = 1 << 4,  // 16 动态 key，当 key 变化时需要完整的 diff 算法做比较
  HYDRATE_EVENTS = 1 << 5,  // 32 表示带有事件监听器的节点
  STABLE_FRAGMENT = 1 << 6,   // 64 一个不会改变子节点顺序的 Fragment
  KEYED_FRAGMENT = 1 << 7, // 128 带有 key 属性的 Fragment
  UNKEYED_FRAGMENT = 1 << 8, // 256 子节点没有 key 的 Fragment
  NEED_PATCH = 1 << 9,   // 512
  DYNAMIC_SLOTS = 1 << 10,  // 动态 solt
  HOISTED = -1,  // 特殊标志是负整数表示永远不会用作 diff
  BAIL = -2 // 一个特殊的标志，指代差异算法
}
复制代码
```

- 静态提升、树结构打平

vue3对不参与更新的元素做静态提升，只会被创建一次，在渲染时直接复用。 作用：减少重复节点的创建，节省内存开销

- 事件监听缓存
- SSR优化

更新类型标记和树结构打平都大大提升了 Vue [SSR 激活](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fscaling-up%2Fssr.html%23client-hydration)的性能表现：

- 单个元素的激活可以基于相应 vnode 的更新类型标记走更快的捷径。
- 在激活时只有区块节点和其动态子节点需要被遍历，这在模板层面上实现更高效的部分激活。

1. **源码体积优化**，移除了一些不常用的API，再就是tree shaking，任何一个函数，比如ref、reactive，只有在用到的时候才进行打包，无用模块都被摇树优化，减少了打包代码体积

```javascript
import {computed,ref} from "vue"
export default defineComponent({
  setup(props,context){
    const age = ref(18)
    let state = reactive({name:"lyllovelemon"})
    const readOnlyAge = computed(()=>age.value++)
    return {
      age,
      state,
      readOnlyAge
    }
  }
})  
复制代码
```

## 虚拟DOM

### 什么是虚拟DOM？如何实现虚拟DOM

React和Vue都使用了**虚拟DOM**技术，虚拟DOM是对真实DOM的一层抽象，它是一颗*js对象树*，用对象的属性描述节点，最后通过渲染器(renderer)将虚拟DOM渲染为真实DOM。

VNode不依赖某一个平台，它可以是浏览器平台，也可以是node平台或者weex平台，这也为前后端同构提供了可能

### 为什么要使用虚拟DOM

一个真实的dom元素，包含的属性和方法是很多的。浏览器在渲染DOM时性能开销很大，频繁渲染DOM最直接的结果就是页面卡顿。

```javascript
// 在控制台输出document
document    
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2f5121dbf6c4c4899a66228784eb93a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

而且浏览器每次收到DOM更新流程时会从头到尾执行一遍更新流程。当你在一次操作时，需要更新10个DOM节点，浏览器没这么智能，收到第一个更新DOM请求后，并不知道后续还有9次更新操作，因此会马上执行流程，最终执行10次流程。

而通过VNode，同样更新10个DOM节点，虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地的一个js对象中，最终将这个js对象一次性attach到DOM树上，避免大量的无谓计算。

*浏览器执行js运算的速度 > 渲染DOM元素的速度*

在vue框架中，渲染流程为:

*数据改变 -> 虚拟DOM ->操作真实DOM-> 视图更新*

### 虚拟DOM是怎么变为真实DOM的

每一次DOM更新流程，Vue会用**patch**函数，对新老节点进行判断，执行创建/销毁节点。通过patchVnode函数和diff算法（参考了snabbdom），新旧Vnode diff比较，**只更新有差异的部分**

在页面首次渲染的时候会调用patch创建新的VNode，不会进行深层次的比较

每个组件对应一个Watcher实例，当数据变化，会触发setter通过notify通知Watcher，对应的Watcher会通知更新并执行更新函数，它会执行render函数获取新的虚拟DOM，然后执行patch比较新旧Vnode，得到有差异的部分；最后根据有差异的部分更新视图

### Diff算法执行过程

vue2版本

1. **只比较同一层级，不跨层比较**

这样的优点是减少了比较次数，算法的事件复杂度降低

1. **比较标签名**

如果同一层级的标签名type不同，直接删除老的VNode

1. **比较key**

如果标签名和key相同，代表新旧VNode是同一个节点

**Diff算法核心：patch，sameVNode，patchVnode,updateChildren**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89880d70662e4c3f902597309fd70072~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

patch源码位置(vue2):vue/blob/main/src/core/vdom/patch.ts

主要逻辑：

1. vnode存在，oldVNode不存在，新增VNode节点
2. vnode不存在，oldVNode存在，删除oldVNode节点
3. 两个都存在，通过sameVnode函数判断是不是同一个节点，如果是，通过patchVnode进行后续对比；不是则把VNode挂载在oldVNode的父元素下，如果组件的根节点被替换，就遍历父节点删除旧节点；如果是服务端渲染就通过hydrating把oldVNode和真实DOM结合

```scss
//vue2版本: vue/blob/main/src/core/vdom/patch.ts    
function patchVnode(
oldVnode,
vnode,
insertedVnodeQueue,
ownerArray,
index,
removeOnly?: any
) {
// 如果新旧节点一致，不做处理直接返回
if (oldVnode === vnode) {
  return
}

if (isDef(vnode.elm) && isDef(ownerArray)) {
  // clone reused vnode
  vnode = ownerArray[index] = cloneVNode(vnode)
}
// 让vnode.el引用到现在的真实dom，当el修改时，vnode.el会同步变化
const elm = (vnode.elm = oldVnode.elm)
// 异步占位符
if (isTrue(oldVnode.isAsyncPlaceholder)) {
  if (isDef(vnode.asyncFactory.resolved)) {
    hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
  } else {
    vnode.isAsyncPlaceholder = true
  }
  return
}

// 如果新旧节点都是静态节点且新旧节点key相同(同一个节点)
// 当vnode是克隆节点且是v-once节点，只需要把oldVnode的组件实例赋给vnode节点
if (
  isTrue(vnode.isStatic) &&
  isTrue(oldVnode.isStatic) &&
  vnode.key === oldVnode.key &&
  (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
) {
  vnode.componentInstance = oldVnode.componentInstance
  return
}

let i
const data = vnode.data
if (isDef(data) && isDef((i = data.hook)) && isDef((i = i.prepatch))) {
  i(oldVnode, vnode)
}

const oldCh = oldVnode.children
const ch = vnode.children
if (isDef(data) && isPatchable(vnode)) {
  // patch比较新旧节点，更新有差异的部分
  for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
  if (isDef((i = data.hook)) && isDef((i = i.update))) i(oldVnode, vnode)
}
if (isUndef(vnode.text)) {
  if (isDef(oldCh) && isDef(ch)) {
    if (oldCh !== ch)
      // 新旧节点都有子节点，则处理比较更新子节点
      updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
  } else if (isDef(ch)) {
    if (__DEV__) {
      // 新节点有子节点且在开发环境，检查重复的key
      checkDuplicateKeys(ch)
    }
    // 旧节点有文本属性，设置为空文本元素
    if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
    addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
  }
  // 旧的子节点存在而新的子节点不存在，删除旧节点
  else if (isDef(oldCh)) {
    removeVnodes(oldCh, 0, oldCh.length - 1)
  } else if (isDef(oldVnode.text)) {
    nodeOps.setTextContent(elm, '')
  }
} 
// 新旧节点文本内容不相同，直接插入新文本内容
else if (oldVnode.text !== vnode.text) {
  nodeOps.setTextContent(elm, vnode.text)
}
if (isDef(data)) {
  if (isDef((i = data.hook)) && isDef((i = i.postpatch))) i(oldVnode, vnode)
}
}
复制代码
```

patchVnode主要做了几个判断：

- 新节点是否是文本节点，如果是，则直接更新dom的文本内容为新节点的文本内容
- 新节点和旧节点如果都有子节点，则处理比较更新子节点
- 只有新节点有子节点，旧节点没有，那么不用比较了，所有节点都是全新的，所以直接全部新建就好了，新建是指创建出所有新DOM，并且添加进父节点
- 只有旧节点有子节点而新节点没有，说明更新后的页面，旧节点全部都不见了，那么要做的，就是把所有的旧节点删除，也就是直接把DOM 删除

再查看**updateChildren**方法

```scss
  function updateChildren(
    parentElm,
    oldCh,
    newCh,
    insertedVnodeQueue,
    removeOnly
  ) {
    // 定义了四个指针：旧前、新前、旧后、新后
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    // 旧前节点
    let oldStartVnode = oldCh[0]
    // 旧后节点
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    // 新前节点
    let newStartVnode = newCh[0]
    // 新后节点
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

 
    const canMove = !removeOnly

    if (__DEV__) {
      checkDuplicateKeys(newCh)
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      // 首先应该不是判断四种命中，而是略过已经加了undefined标记的项
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } 
      // 1.旧前节点与新前节点命中
      else if (sameVnode(oldStartVnode, newStartVnode)) {
        // 精细化比较两个节点 oldStartVnode现在和newStartVnode一样了
        patchVnode(
          oldStartVnode,
          newStartVnode,
          insertedVnodeQueue,
          newCh,
          newStartIdx
        )
        // 移动指针，改变指针指向的节点，这表示这两个节点都处理（比较）完了
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } 
      // 2.旧后与新后命中
      else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(
          oldEndVnode,
          newEndVnode,
          insertedVnodeQueue,
          newCh,
          newEndIdx
        )
        // 移动两个尾指针
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } 
      // 3.旧前与新后命中
      else if (sameVnode(oldStartVnode, newEndVnode)) {
        // Vnode moved right
        patchVnode(
          oldStartVnode,
          newEndVnode,
          insertedVnodeQueue,
          newCh,
          newEndIdx
        )
       // 当新后与旧前命中的时候，此时要移动节点。移动 新后（旧前） 指向的这个节点到老节点的旧后的后面
      // 移动节点：只要插入一个已经在DOM树上 的节点，就会被移动
        canMove &&
          nodeOps.insertBefore(
            parentElm,
            oldStartVnode.elm,
            nodeOps.nextSibling(oldEndVnode.elm)
          )
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } 
      // 4.旧后与新前命中
      else if (sameVnode(oldEndVnode, newStartVnode)) {
        // Vnode moved left
        patchVnode(
          oldEndVnode,
          newStartVnode,
          insertedVnodeQueue,
          newCh,
          newStartIdx
        )
       //当新前与旧后命中的时候，此时要移动节点。移动新前（旧后）指向的这个节点到老节点的 旧前的前面
      // 移动节点：只要插入一个已经在DOM树上的节点，就会被移动
        canMove &&
          nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        if (isUndef(oldKeyToIdx))
          oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) {
          // New element
          createElm(
            newStartVnode,
            insertedVnodeQueue,
            parentElm,
            oldStartVnode.elm,
            false,
            newCh,
            newStartIdx
          )
        } else {
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(
              vnodeToMove,
              newStartVnode,
              insertedVnodeQueue,
              newCh,
              newStartIdx
            )
            oldCh[idxInOld] = undefined
            canMove &&
              nodeOps.insertBefore(
                parentElm,
                vnodeToMove.elm,
                oldStartVnode.elm
              )
          } else {
            // 四种都没有匹配到，key相同但元素不同，创建新元素
            createElm(
              newStartVnode,
              insertedVnodeQueue,
              parentElm,
              oldStartVnode.elm,
              false,
              newCh,
              newStartIdx
            )
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    // 指针移动结束，旧的开始指针>旧的结束指针，代表产生了新元素
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      // 节点新增
      addVnodes(
        parentElm,
        refElm,
        newCh,
        newStartIdx,
        newEndIdx,
        insertedVnodeQueue
      )
    } 
    // 遍历结束，新的开始指针 > 新的结束指针，代表有元素需要被删除
    else if (newStartIdx > newEndIdx) {
      // 节点删除
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }
复制代码
```

**vue3版本的diff**

- vue2是全量diff，vue3是静态标记+非全量diff，节点被打上静态标记就不需要参与diff
- 事件缓存，可以理解为事件为静态的，初始化后就会在更新时从缓存中查找事件
- vue3使用最长递增子序列，主要在patchKeyedChildren函数里

1. 头和头比
2. 尾和尾比
3. 基于最长递增子序列进行新增/更新(移动)/删除

- 老的 children：[ a, b, c, d, e, f, g ]
- 新的 children：[ a, b, f, c, d, e, h, g ]

1. 先进行头和头比，发现不同就结束循环，得到 [ a, b ]
2. 再进行尾和尾比，发现不同就结束循环，得到 [ g ]
3. 再保存没有比较过的节点 [ f, c, d, e, h ]，并通过 newIndexToOldIndexMap 拿到在数组里对应的下标，生成数组 [ 5, 2, 3, 4, -1 ]，-1 是老数组里没有的就说明是新增
4. 然后再拿取出数组里的最长递增子序列，也就是 [ 2, 3, 4 ] 对应的节点 [ c, d, e ]
5. 然后只需要把其他剩余的节点，基于 [ c, d, e ] 的位置进行移动/新增/删除就可以了

```typescript
//packages/runtime-core/src/renderer.ts     
  const patchKeyedChildren = (
    c1: VNode[],
    c2: VNodeArrayChildren,
    container: RendererElement,
    parentAnchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
  ) => {
    let i = 0
    const l2 = c2.length
    let e1 = c1.length - 1 // prev ending index
    let e2 = l2 - 1 // next ending index

    // 1. sync from start
    // (a b) c
    // (a b) d e
    while (i <= e1 && i <= e2) {
      const n1 = c1[i]
      const n2 = (c2[i] = optimized
        ? cloneIfMounted(c2[i] as VNode)
        : normalizeVNode(c2[i]))
      if (isSameVNodeType(n1, n2)) {
        patch(
          n1,
          n2,
          container,
          null,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else {
        break
      }
      i++
    }

    // 2. sync from end
    // a (b c)
    // d e (b c)
    while (i <= e1 && i <= e2) {
      const n1 = c1[e1]
      const n2 = (c2[e2] = optimized
        ? cloneIfMounted(c2[e2] as VNode)
        : normalizeVNode(c2[e2]))
      if (isSameVNodeType(n1, n2)) {
        patch(
          n1,
          n2,
          container,
          null,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else {
        break
      }
      e1--
      e2--
    }

    // 3. common sequence + mount
    // (a b)
    // (a b) c
    // i = 2, e1 = 1, e2 = 2
    // (a b)
    // c (a b)
    // i = 0, e1 = -1, e2 = 0
    if (i > e1) {
      if (i <= e2) {
        const nextPos = e2 + 1
        const anchor = nextPos < l2 ? (c2[nextPos] as VNode).el : parentAnchor
        while (i <= e2) {
          patch(
            null,
            (c2[i] = optimized
              ? cloneIfMounted(c2[i] as VNode)
              : normalizeVNode(c2[i])),
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
          i++
        }
      }
    }

    // 4. common sequence + unmount
    // (a b) c
    // (a b)
    // i = 2, e1 = 2, e2 = 1
    // a (b c)
    // (b c)
    // i = 0, e1 = 0, e2 = -1
    else if (i > e2) {
      while (i <= e1) {
        unmount(c1[i], parentComponent, parentSuspense, true)
        i++
      }
    }

    // 5. unknown sequence
    // [i ... e1 + 1]: a b [c d e] f g
    // [i ... e2 + 1]: a b [e d c h] f g
    // i = 2, e1 = 4, e2 = 5
    else {
      const s1 = i // prev starting index
      const s2 = i // next starting index

      // 5.1 build key:index map for newChildren
      const keyToNewIndexMap: Map<string | number | symbol, number> = new Map()
      for (i = s2; i <= e2; i++) {
        const nextChild = (c2[i] = optimized
          ? cloneIfMounted(c2[i] as VNode)
          : normalizeVNode(c2[i]))
        if (nextChild.key != null) {
          if (__DEV__ && keyToNewIndexMap.has(nextChild.key)) {
            warn(
              `Duplicate keys found during update:`,
              JSON.stringify(nextChild.key),
              `Make sure keys are unique.`
            )
          }
          keyToNewIndexMap.set(nextChild.key, i)
        }
      }

      // 5.2 loop through old children left to be patched and try to patch
      // matching nodes & remove nodes that are no longer present
      let j
      let patched = 0
      const toBePatched = e2 - s2 + 1
      let moved = false
      // used to track whether any node has moved
      let maxNewIndexSoFar = 0
      // works as Map<newIndex, oldIndex>
      // Note that oldIndex is offset by +1
      // and oldIndex = 0 is a special value indicating the new node has
      // no corresponding old node.
      // used for determining longest stable subsequence
      const newIndexToOldIndexMap = new Array(toBePatched)
      for (i = 0; i < toBePatched; i++) newIndexToOldIndexMap[i] = 0

      for (i = s1; i <= e1; i++) {
        const prevChild = c1[i]
        if (patched >= toBePatched) {
          // all new children have been patched so this can only be a removal
          unmount(prevChild, parentComponent, parentSuspense, true)
          continue
        }
        let newIndex
        if (prevChild.key != null) {
          newIndex = keyToNewIndexMap.get(prevChild.key)
        } else {
          // key-less node, try to locate a key-less node of the same type
          for (j = s2; j <= e2; j++) {
            if (
              newIndexToOldIndexMap[j - s2] === 0 &&
              isSameVNodeType(prevChild, c2[j] as VNode)
            ) {
              newIndex = j
              break
            }
          }
        }
        if (newIndex === undefined) {
          unmount(prevChild, parentComponent, parentSuspense, true)
        } else {
          newIndexToOldIndexMap[newIndex - s2] = i + 1
          if (newIndex >= maxNewIndexSoFar) {
            maxNewIndexSoFar = newIndex
          } else {
            moved = true
          }
          patch(
            prevChild,
            c2[newIndex] as VNode,
            container,
            null,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
          patched++
        }
      }

      // 5.3 move and mount
      // generate longest stable subsequence only when nodes have moved
      const increasingNewIndexSequence = moved
        ? getSequence(newIndexToOldIndexMap)
        : EMPTY_ARR
      j = increasingNewIndexSequence.length - 1
      // looping backwards so that we can use last patched node as anchor
      for (i = toBePatched - 1; i >= 0; i--) {
        const nextIndex = s2 + i
        const nextChild = c2[nextIndex] as VNode
        const anchor =
          nextIndex + 1 < l2 ? (c2[nextIndex + 1] as VNode).el : parentAnchor
        if (newIndexToOldIndexMap[i] === 0) {
          // mount new
          patch(
            null,
            nextChild,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
        } else if (moved) {
          // move if:
          // There is no stable subsequence (e.g. a reverse)
          // OR current node is not among the stable sequence
          if (j < 0 || i !== increasingNewIndexSequence[j]) {
            move(nextChild, container, anchor, MoveType.REORDER)
          } else {
            j--
          }
        }
      }
    }
  }
复制代码
```

### 组件是怎么渲染成DOM的

*template->render函数->虚拟DOM-> 真实DOM*

### 源码解析

```kotlin
 // vue2版本源码位置:src/core/vdom/vnode.js
export default class VNode {
  tag: string | void; /*当前节点的标签名*/
  data: VNodeData | void;  /*当前节点对应的对象，包含了具体的一些数据信息，是一个VNodeData类型，可以参考VNodeData类型中的数据信息*/
  children: ?Array<VNode>; /*当前节点的子节点，是一个数组*/
  text: string | void; /*当前节点的文本*/
  elm: Node | void; /*当前虚拟节点对应的真实dom节点*/
  ns: string | void; /*当前节点的命名空间*/
  context: Component | void; /*编译作用域*/
  key: string | number | void;/*节点的key属性，被当作节点的标志，用以优化*/
  functionalContext: Component | void;/*函数化组件作用域*/
  componentOptions: VNodeComponentOptions | void;/*组件的option选项*/
  componentInstance: Component | void; /*当前节点对应的组件的实例*/
  parent: VNode | void; /*当前节点的父节点*/
  raw: boolean; /*简而言之就是是否为原生HTML或只是普通文本，innerHTML的时候为true，textContent的时候为false*/
  isStatic: boolean; /*是否静态节点*/
  isRootInsert: boolean;/*是否作为根节点插入*/
  isComment: boolean; /*是否为注释节点*/
  isCloned: boolean; /*是否为克隆节点*/
  isOnce: boolean;/*是否有v-once指令*/
  constructor (
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions)
  {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.functionalContext = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false}
  }
  get child (): Component | void {
      return this.componentInstance
  }
}
复制代码
```

举个例子，当前我有一颗VNode树，结构如下:

```css
{
    tag: 'div'
    data: {
        class: 'root'
    },
    children: [
        {
            tag: 'span',
            data: {
                class: 'demo'
            }
            text: 'hello,lyllovelemon'
        }
    ]
}
复制代码
```

渲染后变为:

```ini
<div class="root">
    <span class="demo">hello,lyllovelemon</span>
</div>
复制代码
 //vue3版本源码位置：core/packages/runtime-core/src/vnode.ts(3.2.37版本)   
...
export interface VNode<
  HostNode = RendererNode,
  HostElement = RendererElement,
  ExtraProps = { [key: string]: any }
  > {
  // vnode节点标记，判断是否是vnode节点
  __v_isVNode: true
  // 响应式flag标记
  [ReactiveFlags.SKIP]: true
  // 节点类型：vnode节点
  type: VNodeTypes
  // 节点props属性
  props: (VNodeProps & ExtraProps) | null
  // 节点的key属性
  key: string | number | symbol | null
  // 节点的ref属性
  ref: VNodeNormalizedRef | null
  /**
   * SFC only. This is assigned on vnode creation using currentScopeId
   * which is set alongside currentRenderingInstance.
   */
  scopeId: string | null
  /**
   * SFC only. This is assigned to:
   * - Slot fragment vnodes with :slotted SFC styles.
   * - Component vnodes (during patch/hydration) so that its root node can
   *   inherit the component's slotScopeIds
   * @internal
   */
  slotScopeIds: string[] | null

  children: VNodeNormalizedChildren // 当前节点的子节点

  component: ComponentInternalInstance | null   // 当前节点的组件实例
  dirs: DirectiveBinding[] | null
  transition: TransitionHooks<HostElement> | null


  el: HostNode | null // DOM
  anchor: HostNode | null // fragment锚点
  target: HostElement | null // teleport目标元素
  targetAnchor: HostNode | null // teleport目标锚点
  staticCount: number // 静态vnode节点个数
  suspense: SuspenseBoundary | null// suspense
  ssContent: VNode | null
  ssFallback: VNode | null

  // 仅用于optimization
  shapeFlag: number
  patchFlag: number
  dynamicProps: string[] | null // 动态props
  dynamicChildren: VNode[] | null// 动态子节点，需要进行patch

  appContext: AppContext | null // 仅用于应用根节点
  memo?: any[] // v-memo
  isCompatRoot?: true // 仅用于compact
  ce?: (instance: ComponentInternalInstance) => void // 内部自定义元素的拦截hook
}
复制代码
```

**创建虚拟DOM**

```scss
function _createVNode(
  type: VNodeTypes | ClassComponent | typeof NULL_DYNAMIC_COMPONENT,// 节点类型
  props: (Data & VNodeProps) | null = null, // 节点props属性
  children: unknown = null, // 子元素
  patchFlag: number = 0, // patch标志
  dynamicProps: string[] | null = null, // 动态props属性
  isBlockNode = false // 是否为区块节点，区块节点指内部结构稳定的节点
): VNode {
  if (!type || type === NULL_DYNAMIC_COMPONENT) {
    if (__DEV__ && !type) {
      warn(`Invalid vnode type when creating vnode: ${type}.`)
    }
    type = Comment // 注释节点
  }

  if (isVNode(type)) {
    // 创建一个克隆的虚拟DOM，接收3个参数，分别为节点类型、节点props属性、是否合并ref属性
    const cloned = cloneVNode(type, props, true /* mergeRef: true */)
    if (children) {
      normalizeChildren(cloned, children)// 存在子节点，则建立当前虚拟节点与子节点的联系
    }
    // 不为区块节点且当前的节点存在
    if (isBlockTreeEnabled > 0 && !isBlockNode && currentBlock) {
      if (cloned.shapeFlag & ShapeFlags.COMPONENT) {
        currentBlock[currentBlock.indexOf(type)] = cloned
      } else {
        currentBlock.push(cloned)
      }
    }
    cloned.patchFlag |= PatchFlags.BAIL
    return cloned
  }

  // class component normalization.
  if (isClassComponent(type)) {
    type = type.__vccOpts
  }

  // 2.x async/functional component compat
  if (__COMPAT__) {type = convertLegacyComponent(type, currentRenderingInstance) }
  // class & style normalization.
  if (props) {
    // 用于对象的响应式或代理, we need to clone it to enable mutation.
    props = guardReactiveProps(props)!
    let { class: klass, style } = props
    if (klass && !isString(klass)) {
      props.class = normalizeClass(klass) // 标准化class，每一项class都转为字符串处理
    }
    if (isObject(style)) {
      // reactive state objects need to be cloned since they are likely to be
      // mutated
      if (isProxy(style) && !isArray(style)) {
        style = extend({}, style)
      }
      props.style = normalizeStyle(style)// 标准化style
    }
  }

  // encode the vnode type information into a bitmap
  const shapeFlag = isString(type)
    ? ShapeFlags.ELEMENT
    : __FEATURE_SUSPENSE__ && isSuspense(type)
    ? ShapeFlags.SUSPENSE
    : isTeleport(type)
    ? ShapeFlags.TELEPORT
    : isObject(type)
    ? ShapeFlags.STATEFUL_COMPONENT
    : isFunction(type)
    ? ShapeFlags.FUNCTIONAL_COMPONENT
    : 0

  if (__DEV__ && shapeFlag & ShapeFlags.STATEFUL_COMPONENT && isProxy(type)) {
    type = toRaw(type)
    warn(
      `Vue received a Component which was made a reactive object. This can ` +
        `lead to unnecessary performance overhead, and should be avoided by ` +
        `marking the component with `markRaw` or using `shallowRef` ` +
        `instead of `ref`.`,
      `\nComponent that was made reactive: `,
      type
    )
  }

  return createBaseVNode(
    type,
    props,
    children,
    patchFlag,
    dynamicProps,
    shapeFlag,
    isBlockNode,
    true
  )
复制代码
```

**树结构打平**

这里我们引入一个概念“区块”，内部结构是稳定的一个部分可被称之为一个区块。在这个用例中，整个模板只有一个区块，因为这里没有用到任何结构性指令 (比如 v-if 或者 v-for)。

每一个块都会追踪其所有带更新类型标记的后代节点 (不只是直接子节点)，举例来说：

```xml
<div> <!-- root block -->
  <div>...</div>         <!-- 不会追踪 -->
  <div :id="id"></div>   <!-- 要追踪 -->
  <div>                  <!-- 不会追踪 -->
    <div>{{ bar }}</div> <!-- 要追踪 -->
  </div>
</div>
复制代码
```

编译的结果会打平为一个数组，仅包含所有动态的后代节点dynamicChildren

```css
div (block root)
- div 带有 :id 绑定
- div 带有 {{ bar }} 绑定
复制代码
```

当组件需要重渲染时，只需要遍历这个打平的树而不是整棵树，这个过程叫树结构打平，大大减少了虚拟DOM需要遍历的节点数量，模板中任何静态的部分都会被跳过

## 依赖收集与响应式原理

### 什么是响应式

数据变化驱动视图更新就叫响应式

vue2的响应式是通过**Object.defineProperty**实现的，有以下几个问题：

1. 不能深层监听对象的变化
2. 不能获取数组下标和length

```javascript
let val='lyllovelemon'
Object.defineProperty(obj,'name',{
  configurable:true,// 属性是否可配置，默认为false，为true时对应属性可被删除和修改,并且可以通过Object.defineProperty修改
  enumerable:true,// 属性是否可枚举，默认为false，为true是属性可被for...in或者Object.keys遍历
  writable:true,// 属性是否可写，默认为false，为true时属性值才可改变 
  value:'lyllovelemon',// 属性的初始值，可以是任何有效的JavaScript值(数值、对象、函数等)，默认为undefined
  get(){
    return val
  },// 属性读取
  set(newVal){
    val = newVal
  }// 属性写入
})
console.log(obj.name)// 'lyllovelemon',表示属性的value已生效
obj.name='lyl'
console.log(obj.name)// 'lyl',getter和setter都已生效
复制代码
```

vue3使用ES6的**proxy**实现响应式，proxy是支持数组的，因此解决了无法获取数组下标和length的问题；对于深层监听也不需要使用递归解决，当get判断值为对象时，将对象响应式处理即可

### 原理实现

**vue3源码位置：packages/reactivity/src/reactive.ts**

```typescript
export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
export function reactive(target: object) {
  // 监听元素为只读的，不做处理返回
  if (isReadonly(target)) {
    return target
  }
  // 否则通过createReactiveObject处理
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers,
    reactiveMap
  )
}
复制代码
```

响应式实际是在createReactiveObject函数中处理的，该函数主要做了几件事：

- **判断监听**
- **实例化proxy对象**

```typescript
function createReactiveObject(
  target: Target, // 监听的元素
  isReadonly: boolean, // 是否只读
  baseHandlers: ProxyHandler<any>, // proxy处理函数
  collectionHandlers: ProxyHandler<any>,// 收集变化函数
  proxyMap: WeakMap<Target, any> // proxyMap 
) {
  // 监听元素不是对象则返回该元素
  if (!isObject(target)) {
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`)
    }
    return target
  }
  // 监听元素是原生且不是只读元素，是响应式的则返回该元素
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }
  // 监听元素已经被Proxy处理过了，从proxyMap中查找该元素并返回
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  // 获取元素类型
  const targetType = getTargetType(target)
  // 监听元素为基本数据类型则返回该元素
  if (targetType === TargetType.INVALID) {
    return target
  }
  // new实例化proxy对象
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  // 将监听元素加入到proxyMap并返回监听元素
  proxyMap.set(target, proxy)
  return proxy
}
复制代码
```

**getTargetType**函数主要用于处理元素类型

```css
function getTargetType(value: Target) {
  return value[ReactiveFlags.SKIP] || !Object.isExtensible(value)
    ? TargetType.INVALID
    : targetTypeMap(toRawType(value))
}
复制代码
//TargetType常量定义，基本数据类型为INVALID，object/array为COMMON
// Map/Set/WeakMap/WeakSet为COLLECTION
const enum TargetType {
  INVALID = 0,
  COMMON = 1,
  COLLECTION = 2
}

function targetTypeMap(rawType: string) {
  switch (rawType) {
    case 'Object':
    case 'Array':
      return TargetType.COMMON
    case 'Map':
    case 'Set':
    case 'WeakMap':
    case 'WeakSet':
      return TargetType.COLLECTION
    default:
      return TargetType.INVALID
  }
}
复制代码
```

## 事件机制原理

### Vue事件机制API

vue提供了四个事件机制API,分别是:on,on,on,off,emit,emit,emit,once

### 初始化事件

vue2版本源码位置:vue/blob/main/src/core/instance/events.ts

核心方法为initEvents，主要作用：

- 初始化events和hasHookEvent变量
- 父组件绑定的事件存在就调用updateComponentListeners，这个函数里调用updateListeners，接收新旧事件对象作为参数，遍历新事件对象，标准化事件，事件绑定了once就调用对应逻辑；旧事件没有新事件有，创建事件对象，旧事件有新事件没有，删除对应旧事件；新旧事件不一致使用新事件

```javascript
export function initEvents(vm: Component) {
  // 给vm实例绑定_events属性，值为null
  vm._events = Object.create(null)
  // 给vm实例绑定_hasHookEvent实例，标志是否存在钩子
  vm._hasHookEvent = false
  // 初始化父组件绑定的事件
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
复制代码
```

### $on

on用于自定义一个事件，通过on用于自定义一个事件，通过on用于自定义一个事件，通过emit触发

```csharp
// 接收2个参数，分别为事件名（事件数组）,需要执行的函数
Vue.prototype.$on = function (
    event: string | Array<string>,
    fn: Function
  ): Component {
    const vm: Component = this
    // 如果events是数组，遍历执行，给每一项事件绑定fn函数
    if (isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$on(event[i], fn)
      }
    } else {
      // events是单个，建立一个空数组，将fn函数push进数组
      ;(vm._events[event] || (vm._events[event] = [])).push(fn)
      // hook性能优化专用
      if (hookRE.test(event)) {
        vm._hasHookEvent = true
      }
    }
    return vm
  }
复制代码
```

### $off

$off用于移除自定义事件

```csharp
//接收2个可选参数，分别为事件名（事件数组）,需要执行的函数
Vue.prototype.$off = function (
    event?: string | Array<string>,
    fn?: Function
  ): Component {
    const vm: Component = this
    // off函数未传参，将事件置空，返回vm实例
    if (!arguments.length) {
      vm._events = Object.create(null)
      return vm
    }
    // events是数组，遍历每一项解除函数绑定，返回vm实例
    if (isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$off(event[i], fn)
      }
      return vm
    }
    // specific event
    const cbs = vm._events[event!]
    if (!cbs) {
      return vm
    }
    // fn未传参，清空events数组
    if (!fn) {
      vm._events[event!] = null
      return vm
    }
    // 特别处理cbs，遍历每一项移除
    let cb
    let i = cbs.length
    while (i--) {
      cb = cbs[i]
      if (cb === fn || cb.fn === fn) {
        cbs.splice(i, 1)
        break
      }
    }
    return vm
  }
复制代码
```

### $emit

$emit触发自定义事件

```csharp
 Vue.prototype.$emit = function (event: string): Component {
    const vm: Component = this
    if (__DEV__) {
      const lowerCaseEvent = event.toLowerCase()
      if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
        tip(
          `Event "${lowerCaseEvent}" is emitted in component ` +
            `${formatComponentName(
              vm
            )} but the handler is registered for "${event}". ` +
            `Note that HTML attributes are case-insensitive and you cannot use ` +
            `v-on to listen to camelCase events when using in-DOM templates. ` +
            `You should probably use "${hyphenate(
              event
            )}" instead of "${event}".`
        )
      }
    }
    // 从vm的_events属性中获取cbs
    let cbs = vm._events[event]
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs
      // cbs类数组转数组
      const args = toArray(arguments, 1)
      const info = `event handler for "${event}"`
      // 遍历cbs每一项调用invokeWithErrorHandling，向上冒泡捕获错误
      for (let i = 0, l = cbs.length; i < l; i++) {
        invokeWithErrorHandling(cbs[i], vm, args, vm, info)
      }
    }
    return vm
  }
复制代码
```

### $once

$once自定义一个只执行一次的事件，执行过后自动移除该事件

```csharp
// 接收两个参数，分别为事件名和需要执行的函数
Vue.prototype.$once = function (event: string, fn: Function): Component {
    const vm: Component = this
    // 内部定义on方法，做2件事:
    // 1.在第1次执行的时候将事件销毁
   // 2. 将实例绑定到fn函数
    function on() {
      vm.$off(event, on)
      fn.apply(vm, arguments)
    }
    on.fn = fn
    vm.$on(event, on)
    return vm
  }
复制代码
```

### 事件缓存

vue3在vue2的基础上增加了事件监听缓存

## 模板编译原理

### template函数怎么编译成render函数的

**vue2版本**创建vue实例时，组件通过 **_init**方法进行**初始化，** 这个方法主要用于初始化data,method,生命周期等属性，最后调用$mount进行实例挂载

```typescript
 //vue2.7.10源码地址：src/core/instance/init.ts
export function initMixin(Vue: typeof Component) {
  Vue.prototype._init = function (options?: Record<string, any>) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (__DEV__ && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to mark this as a Vue instance without having to do instanceof
    // check
    vm._isVue = true
    // avoid instances from being observed
    vm.__v_skip = true
    // effect scope
    vm._scope = new EffectScope(true /* detached */)
    vm._scope._vm = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options as any)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor as any),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (__DEV__) {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate', undefined, false /* setContext */)
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (__DEV__ && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el) // 实例挂载
    }
  }
}                                        
复制代码
```

查看$mount方法，render函数不存在，将template通过compilerToFunctions方法编译成得到AST，render和staticRenderFns（vue的编译优化，静态节点不需要patch），render函数在运行时会返回虚拟DOM 核心方法就是**compileToFunctions**

```js
//src/platforms/web/runtime-with-compiler.ts
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    __DEV__ &&
      warn(
        `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
      )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (__DEV__ && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (__DEV__) {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      // @ts-expect-error
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (__DEV__ && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(
        template,
        {
          outputSourceRange: __DEV__,
          shouldDecodeNewlines,
          shouldDecodeNewlinesForHref,
          delimiters: options.delimiters,
          comments: options.comments
        },
        this
      )
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (__DEV__ && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}
复制代码
```

**compileToFunction**是在createCompiler方法中被调用的，createCompiler调用了parse进行模板编译

```js
//src/platforms/web/compiler/index.ts
const { compile, compileToFunctions } = createCompiler(baseOptions)

export { compile, compileToFunctions }

//src/platforms/web/compiler/index.ts
export const createCompiler = createCompilerCreator(function baseCompile(
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)//1.parse模板解析
  if (options.optimize !== false) {
    optimize(ast, options)// 2.optimize 优化
  }
  const code = generate(ast, options)//3.generate 代码生成
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
复制代码
```

**template -> AST -> render**

1. **parse**方法主要用于处理template，通过正则表达式解析template模板中的指令、class、style等数据，生成AST
2. **optimiz**e主要用于标记static静态节点和静态根节点，这是vue编译时进行的优化，优点是只有在挂载的时候生成，静态节点不会参与后续的diff，也就是说视图更新时，会跳过静态节点的比较，减少了节点的比较次数，diff性能得到了提升
3. **generate**，主要用于将AST转换为render function字符串，得到render字符串和staticRenderFns字符串

```js
export function generate(
ast: ASTElement | void,
options: CompilerOptions
): CodegenResult {
  const state = new CodegenState(options)
  const code = ast
    ? ast.tag === 'script'
      ? 'null'
      : genElement(ast, state)
    : '_c("div")'
  return {
    render: `with(this){return ${code}}`,
    staticRenderFns: state.staticRenderFns
  }
}
复制代码
```

## Transition实现原理

### Transition组件基本使用

[Transition组件](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fbuilt-ins%2Ftransition.html%23the-transition-component)是vue提供的内置组件，用于制作状态变化的动画

会在组件进入或离开DOM时应用动画

```xml
<Transition name="fade" mode="out-in">
  <p v-if="show">hello,lyllovelemon</p>
</Transition>
  
<style  scoped>
 .fade-enter-active,.fade-leave-active{
     transition:opacity 0.5s ease;
 }
 .fade-enter-from,.fade-leave-to{
     opacity: 0;
 }
</style>    
复制代码
```

Transition组件插槽只支持单个元素，当插槽内容时组件时，必须确保组件只有一个根元素，否则会报错 expects exactly one child element or component.

[TransitionGroup](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fbuilt-ins%2Ftransition-group.html)

用于v-for列表元素、组件的插入、移除、改变添加动画效果

```xml
<template>
        <button @click="addItem">任意位置添加一项</button>
        <button @click="reset">重置</button>
        <button @click="shuffleItem">打乱</button>
        <TransitionGroup name="list" tag="ul">
            <li v-for="item in list" :key="item">{{item}}
                <button @click="remove(item)">删除</button>
            </li>
        </TransitionGroup>
</template>
<script>
    import {ref,defineComponent} from 'vue'
    import {shuffle} from 'lodash-es'
    export default defineComponent({
        setup(){
            const getInitialItems=()=>[1,2,3,4,5]
            const list = ref(getInitialItems())
            let id = list.value.length + 1
            const addItem=()=>{
                const i = Math.round(Math.random()*list.value.length)
                list.value.splice(i,0,id++)
            }
            const reset=()=>{
                list.value=getInitialItems()
            }
            const shuffleItem=()=>{
                list.value = shuffle(list.value)
            }
            const remove=(item)=>{
                const index = list.value.indexOf(item)
                index>-1 && list.value.splice(index,1)
            }
            return{
                list,
                shuffleItem,
                addItem,
                reset,
                remove
            }
        }
    })

</script>
<style  scoped>
  // 声明过渡效果
 .list-move,
 .list-enter-active,
 .list-leave-active {
    transition: all 0.5s ease;
 }
 // 声明进入和离开的状态 
 .list-enter-from,
 .list-leave-to {
    opacity: 0;
    transform: translateX(30px);
 }
// 确保离开的项被移出了布局流,以便正确计算移动时的动画效果  
.list-leave-active{
    position: absolute;
}
</style>
复制代码
```

### Transition组件实现原理

源码位置(vue3)：packages/runtime-dom/src/components/Transition.ts

原理也很简单，Transition既然是vue组件，当然也会经历虚拟DOM转换为真实DOM。Transtion组件的特殊之处是为渲染子节点的VNode添加key属性，然后在子节点的VNode下添加transition属性，表示这是个transition组件渲染的VNode，然后在转换为真实DOM的时候特殊处理

```typescript
// 导出Transition组件常量，它是一个函数组件类型
// 接收参数分别为props,slots
// 返回一个h函数
export const Transition: FunctionalComponent<TransitionProps> = (
  props,
  { slots }
) => h(BaseTransition, resolveTransitionProps(props), slots)

// TransitionProps是一个接口类型
export interface TransitionProps extends BaseTransitionProps<Element> {
  // 过渡效果命名，用于基于css的过渡效果
  name?: string
  // 类型，TRANSITION或者ANIMATION
  type?: typeof TRANSITION | typeof ANIMATION
  // 是否支持css过渡效果
  css?: boolean
  // 动画持续时长,接收数值类型，单位毫秒；也可以单独定义进入/离开动画的持续时长
  duration?: number | { enter: number; leave: number }
  // 自定义transition classes,相比vue2的6个增加了3个
  enterFromClass?: string
  enterActiveClass?: string
  enterToClass?: string
  appearFromClass?: string
  appearActiveClass?: string
  appearToClass?: string
  leaveFromClass?: string
  leaveActiveClass?: string
  leaveToClass?: string
}
复制代码
```

查看TransitionProps继承的BaseTransitionProps，这个接口主要定义了**mode**和一些**JavaScript钩子**

```typescript
// packages/runtime-core/src/components/BaseTransition.ts    
export interface BaseTransitionProps<HostElement = RendererElement> {
  // 过渡模式，支持in-out/out-in/default
  mode?: 'in-out' | 'out-in' | 'default'
  // 是否首次渲染
  appear?: boolean
  // 是否通过自定义指令(v-show等)控制
  // If true, indicates this is a transition that doesn't actually insert/remove
  // the element, but toggles the show / hidden status instead.
  // The transition hooks are injected, but will be skipped by the renderer.
  // Instead, a custom directive can control the transition by calling the
  // injected hooks (e.g. v-show).
  persisted?: boolean

  // 进入DOM的JavaScript钩子，可以通过@before-enter="xxx"形式调用 
  onBeforeEnter?: Hook<(el: HostElement) => void>
  onEnter?: Hook<(el: HostElement, done: () => void) => void>
  onAfterEnter?: Hook<(el: HostElement) => void>
  onEnterCancelled?: Hook<(el: HostElement) => void>
  // leave Javascript钩子
  onBeforeLeave?: Hook<(el: HostElement) => void>
  onLeave?: Hook<(el: HostElement, done: () => void) => void>
  onAfterLeave?: Hook<(el: HostElement) => void>
  onLeaveCancelled?: Hook<(el: HostElement) => void> // only fired in persisted mode
  // appear Javascript钩子
  onBeforeAppear?: Hook<(el: HostElement) => void>
  onAppear?: Hook<(el: HostElement, done: () => void) => void>
  onAfterAppear?: Hook<(el: HostElement) => void>
  onAppearCancelled?: Hook<(el: HostElement) => void>
}
复制代码
```

由此Transition组件的props已经清晰，再回到transition对应的源码位置,核心方法为resolveTransitionProps

```scss
// Transition返回了一个h函数，接收参数分别是Transition的props，需要执行的函数，插槽
export const Transition: FunctionalComponent<TransitionProps> = (
  props,
  { slots }
) => h(BaseTransition, resolveTransitionProps(props), slots)

export function resolveTransitionProps(
  rawProps: TransitionProps
): BaseTransitionProps<Element> {
  const baseProps: BaseTransitionProps<Element> = {}
  // 遍历Transition组件的每一项props，没有定义在DOMTransitionPropsValidators中的属性一律转换为any类型
  for (const key in rawProps) {
    if (!(key in DOMTransitionPropsValidators)) {
      ;(baseProps as any)[key] = (rawProps as any)[key]
    }
  }
  // css属性为false，直接返回baseProps
  if (rawProps.css === false) {
    return baseProps
  }

  // 解构一些基本的属性，包括9个自定义class，name未赋值定义为v
  const {
    name = 'v',
    type,
    duration,
    enterFromClass = `${name}-enter-from`,
    enterActiveClass = `${name}-enter-active`,
    enterToClass = `${name}-enter-to`,
    appearFromClass = enterFromClass,
    appearActiveClass = enterActiveClass,
    appearToClass = enterToClass,
    leaveFromClass = `${name}-leave-from`,
    leaveActiveClass = `${name}-leave-active`,
    leaveToClass = `${name}-leave-to`
  } = rawProps

  // 分别处理enter，appear，leave三种情况的class
  const legacyClassEnabled =
    __COMPAT__ &&
    compatUtils.isCompatEnabled(DeprecationTypes.TRANSITION_CLASSES, null)
  let legacyEnterFromClass: string
  let legacyAppearFromClass: string
  let legacyLeaveFromClass: string
  if (__COMPAT__ && legacyClassEnabled) {
    const toLegacyClass = (cls: string) => cls.replace(/-from$/, '')
    if (!rawProps.enterFromClass) {
      legacyEnterFromClass = toLegacyClass(enterFromClass)
    }
    if (!rawProps.appearFromClass) {
      legacyAppearFromClass = toLegacyClass(appearFromClass)
    }
    if (!rawProps.leaveFromClass) {
      legacyLeaveFromClass = toLegacyClass(leaveFromClass)
    }
  }

  // 标准化动画持续时长，拿到进入/离开DOM持续时长
  const durations = normalizeDuration(duration)
  const enterDuration = durations && durations[0]
  const leaveDuration = durations && durations[1]
  // 解构拿到JavaScript钩子方法
  const {
    onBeforeEnter,
    onEnter,
    onEnterCancelled,
    onLeave,
    onLeaveCancelled,
    onBeforeAppear = onBeforeEnter,
    onAppear = onEnter,
    onAppearCancelled = onEnterCancelled
  } = baseProps

  // 处理进入动画的完成，移除对应的enter to class和enter active class
  const finishEnter = (el: Element, isAppear: boolean, done?: () => void) => {
    removeTransitionClass(el, isAppear ? appearToClass : enterToClass)
    removeTransitionClass(el, isAppear ? appearActiveClass : enterActiveClass)
    done && done()
  }

  // 处理离开动画的完成，移除对应的leave from class，leave to class，leave active class
  const finishLeave = (
    el: Element & { _isLeaving?: boolean },
    done?: () => void
  ) => {
    el._isLeaving = false
    removeTransitionClass(el, leaveFromClass)
    removeTransitionClass(el, leaveToClass)
    removeTransitionClass(el, leaveActiveClass)
    done && done()
  }

  const makeEnterHook = (isAppear: boolean) => {
    return (el: Element, done: () => void) => {
      const hook = isAppear ? onAppear : onEnter
      const resolve = () => finishEnter(el, isAppear, done)
      callHook(hook, [el, resolve])
      // 封装了requestAnimationFrame方法
      nextFrame(() => {
        removeTransitionClass(el, isAppear ? appearFromClass : enterFromClass)
        if (__COMPAT__ && legacyClassEnabled) {
          removeTransitionClass(
            el,
            isAppear ? legacyAppearFromClass : legacyEnterFromClass
          )
        }
        addTransitionClass(el, isAppear ? appearToClass : enterToClass)
        if (!hasExplicitCallback(hook)) {
          whenTransitionEnds(el, type, enterDuration, resolve)
        }
      })
    }
  }

  return extend(baseProps, {
    onBeforeEnter(el) {
      callHook(onBeforeEnter, [el])
      addTransitionClass(el, enterFromClass)
      if (__COMPAT__ && legacyClassEnabled) {
        addTransitionClass(el, legacyEnterFromClass)
      }
      addTransitionClass(el, enterActiveClass)
    },
    onBeforeAppear(el) {
      callHook(onBeforeAppear, [el])
      addTransitionClass(el, appearFromClass)
      if (__COMPAT__ && legacyClassEnabled) {
        addTransitionClass(el, legacyAppearFromClass)
      }
      addTransitionClass(el, appearActiveClass)
    },
    onEnter: makeEnterHook(false),
    onAppear: makeEnterHook(true),
    onLeave(el: Element & { _isLeaving?: boolean }, done) {
      el._isLeaving = true
      const resolve = () => finishLeave(el, done)
      addTransitionClass(el, leaveFromClass)
      if (__COMPAT__ && legacyClassEnabled) {
        addTransitionClass(el, legacyLeaveFromClass)
      }
      // force reflow so *-leave-from classes immediately take effect (#2593)
      forceReflow()
      addTransitionClass(el, leaveActiveClass)
      nextFrame(() => {
        if (!el._isLeaving) {
          // cancelled
          return
        }
        removeTransitionClass(el, leaveFromClass)
        if (__COMPAT__ && legacyClassEnabled) {
          removeTransitionClass(el, legacyLeaveFromClass)
        }
        addTransitionClass(el, leaveToClass)
        if (!hasExplicitCallback(onLeave)) {
          whenTransitionEnds(el, type, leaveDuration, resolve)
        }
      })
      callHook(onLeave, [el, resolve])
    },
    onEnterCancelled(el) {
      finishEnter(el, false)
      callHook(onEnterCancelled, [el])
    },
    onAppearCancelled(el) {
      finishEnter(el, true)
      callHook(onAppearCancelled, [el])
    },
    onLeaveCancelled(el) {
      finishLeave(el)
      callHook(onLeaveCancelled, [el])
    }
  } as BaseTransitionProps<Element>)
}
复制代码
```

## keepAlive原理

### 基本使用

[keepAlive](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fbuilt-ins%2Fkeep-alive.html%23basic-usage)是一个内置组件，主要用于组件缓存，它包裹的组件在切换后不会被销毁，而是保留在内存中，避免重复渲染DOM。**include/exclude**用于包含/排除组件，**max**用于限制最大缓存实例个数，它使用的是LRU算法

LRU缓存(最大最少使用缓存):缓存的实例个数超过最大数量，最久没被访问的缓存实例将被销毁，以便为新实例腾出空间

缓存实例的生命周期

**onActivated**：组件被挂载时调用

**onDeactivated**:组件被卸载时调用

```xml
<script setup>
  import {onActivated,onDeactivated} from "vue"
  onActivated(()=>{
   // 调用时机为首次挂载
  // 以及每次从缓存中被重新插入时
  })
  onDeactivated(()=>{
    // 在从 DOM 上移除、进入缓存
  // 以及组件卸载时调用
  })
</script>	
复制代码
```

### keepAlive实现原理

```scss
// vue3版本源码位置:packages/runtime-core/src/components/KeepAlive.ts
// MatchPattern类，支持传入字符串、正则表达式或者字符串数组、正则数组
type MatchPattern = string | RegExp | (string | RegExp)[]

// 定义KeepAliveProps接口，接收三个参数
export interface KeepAliveProps {
  // include缓存白名单实例，可选参数
  include?: MatchPattern
  // exclude缓存黑名单实例，可选参数
  exclude?: MatchPattern
  // 最大可缓存个数，可选参数，支持传入数字或字符串
  max?: number | string
}
type CacheKey = string | number | symbol | ConcreteComponent
// 定义Cache类，用于缓存虚拟DOM，Map类型
type Cache = Map<CacheKey, VNode>
// 定义Keys类，用于缓存虚拟DOM对应的key，Set类型
type Keys = Set<CacheKey>

// KeepAliveContext接口
export interface KeepAliveContext extends ComponentRenderContext {
  // 渲染器实例
  renderer: RendererInternals
  // 组件创建实例方法
  activate: (
    vnode: VNode,
    container: RendererElement,
    anchor: RendererNode | null,
    isSVG: boolean,
    optimized: boolean
  ) => void
  // 组件销毁实例方法
  deactivate: (vnode: VNode) => void
}

const KeepAliveImpl: ComponentOptions = {
  name: `KeepAlive`,
  // 标志用于摇树优化
  __isKeepAlive: true,
  // 定义的props，前面已说过不再重复
  props: {
    include: [String, RegExp, Array],
    exclude: [String, RegExp, Array],
    max: [String, Number]
  },

  // 在setup生命周期执行
  setup(props: KeepAliveProps, { slots }: SetupContext) {
    // 获取当前正在渲染的实例
    const instance = getCurrentInstance()!
    // 取实例的上下文作为共享上下文(摇树优化用)
    const sharedContext = instance.ctx as KeepAliveContext
    // 如果内部渲染实例未注册，表明是服务端渲染，在keepAlive内部渲染子组件
    if (__SSR__ && !sharedContext.renderer) {
      return () => {
        const children = slots.default && slots.default()
        return children && children.length === 1 ? children[0] : children
      }
    }
    // cache用于缓存虚拟DOM，keys缓存虚拟DOM对应的key
    const cache: Cache = new Map()
    const keys: Keys = new Set()
    // 当前渲染的虚拟DOM，初始化时为null
    let current: VNode | null = null

    //开发环境且开启了devtools，实例增加__v_cache属性使用缓存
    if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
      ;(instance as any).__v_cache = cache
    }
    // 获取实例的suspense属性
    const parentSuspense = instance.suspense

    // 从共享上下文中解构出patch,move,unmount,createElement方法
    const {
      renderer: {
        p: patch,
        m: move,
        um: _unmount,
        o: { createElement }
      }
    } = sharedContext
    // 创建div元素
    const storageContainer = createElement('div')

    // 共享上下文的activate属性定义组件被创建的方法
    sharedContext.activate = (vnode, container, anchor, isSVG, optimized) => {
      // 从虚拟DOM的component属性获取实例
      const instance = vnode.component!
      move(vnode, container, anchor, MoveType.ENTER, parentSuspense)
      // 调用patch方法进行新旧VNode比较
      patch(
        instance.vnode,
        vnode,
        container,
        anchor,
        instance,
        parentSuspense,
        isSVG,
        vnode.slotScopeIds,
        optimized
      )
      queuePostRenderEffect(() => {
        instance.isDeactivated = false
        if (instance.a) {
          invokeArrayFns(instance.a)
        }
        const vnodeHook = vnode.props && vnode.props.onVnodeMounted
        if (vnodeHook) {
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
      }, parentSuspense)

      if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
        // 更新组件树
        devtoolsComponentAdded(instance)
      }
    }
// 共享上下文的deactivate属性定义组件被销毁的方法
    sharedContext.deactivate = (vnode: VNode) => {
      const instance = vnode.component!
      move(vnode, storageContainer, null, MoveType.LEAVE, parentSuspense)
      queuePostRenderEffect(() => {
        if (instance.da) {
          invokeArrayFns(instance.da)
        }
        const vnodeHook = vnode.props && vnode.props.onVnodeUnmounted
        if (vnodeHook) {
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
        instance.isDeactivated = true
      }, parentSuspense)

      if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
        // 更新组件树
        devtoolsComponentAdded(instance)
      }
    }

    function unmount(vnode: VNode) {
      // 重置shapeFlag用于unmounted
      resetShapeFlag(vnode)
      _unmount(vnode, instance, parentSuspense, true)
    }

    function pruneCache(filter?: (name: string) => boolean) {
      // 遍历已缓存的虚拟DOM map
      cache.forEach((vnode, key) => {
        // 获取当前组件名
        const name = getComponentName(vnode.type as ConcreteComponent)
        if (name && (!filter || !filter(name))) {
          pruneCacheEntry(key)
        }
      })
    }

    function pruneCacheEntry(key: CacheKey) {
      // 从缓存的虚拟DOM map中取出key对应的VNode
      const cached = cache.get(key) as VNode
      // 当前没有被渲染的实例(旧的VNode需要被删除)或者
      //缓存VNode的type不等于当前实例的type(新旧VNode不相等)
      if (!current || cached.type !== current.type) {
        // unmount卸载VNode
        unmount(cached)
      } else if (current) {
        // 当前激活实例不再使用keep-alive，重置flag
        resetShapeFlag(current)
      }
      // 从cache和keys中删除对应实例
      cache.delete(key)
      keys.delete(key)
    }

    // watch include/exclude属性变更
    watch(
      () => [props.include, props.exclude],
      ([include, exclude]) => {
        // include命中则增加
        include && pruneCache(name => matches(include, name))
        // exclude命中则删除
        exclude && pruneCache(name => !matches(exclude, name))
      },
      // prune post-render after `current` has been updated
      { flush: 'post', deep: true }
    )

    // 缓存渲染后的子树
    let pendingCacheKey: CacheKey | null = null
    const cacheSubtree = () => {
      // fix #1621, the pendingCacheKey could be 0
      if (pendingCacheKey != null) {
        cache.set(pendingCacheKey, getInnerChild(instance.subTree))
      }
    }
    onMounted(cacheSubtree)
    onUpdated(cacheSubtree)

    onBeforeUnmount(() => {
      cache.forEach(cached => {
        // 解构获取subTree,suspense
        const { subTree, suspense } = instance
        // 获取子树的child vnode
        const vnode = getInnerChild(subTree)
        if (cached.type === vnode.type) {
          // 重置标志
          resetShapeFlag(vnode)
          // 触发deactivated hook
          const da = vnode.component!.da
          da && queuePostRenderEffect(da, suspense)
          return
        }
        unmount(cached)
      })
    })

    return () => {
      pendingCacheKey = null
      // 没有插槽则返回
      if (!slots.default) {
        return null
      }

      const children = slots.default()
      // 获取keepAlive第一个子节点的VNode
      const rawVNode = children[0]
      // keepAlive内部只允许包裹一个组件
      if (children.length > 1) {
        if (__DEV__) {
          warn(`KeepAlive should contain exactly one component child.`)
        }
        current = null
        return children
      } else if (
        // 包裹的组件不是VNode，也不是有状态的组件或者Suspense，直接返回当前子组件
        !isVNode(rawVNode) ||
        (!(rawVNode.shapeFlag & ShapeFlags.STATEFUL_COMPONENT) &&
          !(rawVNode.shapeFlag & ShapeFlags.SUSPENSE))
      ) {
        current = null
        return rawVNode
      }
      // 获取包裹组件的子节点VNode
      let vnode = getInnerChild(rawVNode)
      // 获取vnode的type属性，它是ConcreteComponent
      const comp = vnode.type as ConcreteComponent
      // 异步组件需要基于已加载的内部组件进行命名检查
      const name = getComponentName(
        isAsyncWrapper(vnode)
          ? (vnode.type as ComponentOptions).__asyncResolved || {}
          : comp
      )
      // 解构include，exclude，max
      const { include, exclude, max } = props

      // 定义了include属性且命名不匹配
      // 或定义了exclude属性且命名匹配
      if (
        (include && (!name || !matches(include, name))) ||
        (exclude && name && matches(exclude, name))
      ) {
        // 当前节点为VNode，返回keepAlive第一层子组件
        current = vnode
        return rawVNode
      }
      // vnode没有key属性，key为comp，否则取vnode.key
      const key = vnode.key == null ? comp : vnode.key
      // 根据key获取缓存对应的虚拟DOM
      const cachedVNode = cache.get(key)

      if (vnode.el) {
        // 克隆vnode节点
        vnode = cloneVNode(vnode)
        // 是Suspense组件，给原生VNode赋值ssContent属性，值为vnode
        if (rawVNode.shapeFlag & ShapeFlags.SUSPENSE) {
          rawVNode.ssContent = vnode
        }
      }
      // 在beforeMount/beforeUpdate钩子中缓存到instance.subTree
      pendingCacheKey = key

      if (cachedVNode) {
        // 已缓存的vNode存在就把el和component赋值到vnode
        vnode.el = cachedVNode.el
        vnode.component = cachedVNode.component
        if (vnode.transition) {
          // 递归更新子树的transition hook
          setTransitionHooks(vnode, vnode.transition!)
        }
        // 避免vnode作为新元素挂载
        vnode.shapeFlag |= ShapeFlags.COMPONENT_KEPT_ALIVE
        // keys set中先删后加，确保是最新的key
        keys.delete(key)
        keys.add(key)
      } else {
        // 新增key
        keys.add(key)
        // max属性存在且当前已缓存的keys容量大于max
        if (max && keys.size > parseInt(max as string, 10)) {
          // 使用LRU更新key
          pruneCacheEntry(keys.values().next().value)
        }
      }
      // 避免vnode被unmounted
      vnode.shapeFlag |= ShapeFlags.COMPONENT_SHOULD_KEEP_ALIVE

      current = vnode
      // 是否为Suspense组件，是返回原生Vnode，否返回vnode
      return isSuspense(rawVNode.type) ? rawVNode : vnode
    }
  }
}
export const isSuspense = (type: any): boolean => type.__isSuspense
复制代码
```

**keepAlive是在哪个生命周期被调用的**

**vue2**

- 在**created**阶段，初始化cache、keys，cache用于缓存虚拟DOM，是一个map集合。keys用于缓存组件的key集合，是一个Set
- 在**mounted**阶段，监听include，exclude的变化，执行相应操作
- **destroyed**阶段，删除所有缓存相关实例

vue3

## Supsense原理与异步

### 基本使用

[**suspense**](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fbuilt-ins%2Fsuspense.html%23async-dependencies)是vue的一个内置组件，用于处理异步依赖,它有两个插槽， **#default**和 **#fallback**,每个插槽只允许一个直接的子节点

```xml
<Suspense>
  <!--1.初始渲染时，渲染#default插槽的内容，如果在这个过程遇到异步依赖，会进入挂起状态-->
  <Container />
  <!--2.在挂起状态时，展示的是#fallback插槽的内容--> 
  <!--3.当所有异步依赖完成后，suspense进入完成状态，展示#default插槽的内容-->  
  <template #fallback>
    loading...
  </template>
</Suspense>
复制代码
```

### Suspense实现原理

```bash
//vue3源码位置：packages/runtime-core/src/components/Suspense.ts
复制代码
```

## Teleport是怎么实现选择性挂载的

[Teleport](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fbuilt-ins%2Fteleport.html)是vue的一个内置组件，类似于React的Portal，它可以让组件渲染在父组件以外的DOM上，主要支持to和disabled两个参数

**to**  必选，Teleport目标挂载的DOM元素

**disabled** 可选，用于禁用Teleport的功能，插槽内容不会移动到任何位置

使用场景：弹窗

```vue
 // index.vue
<template>
    <div class="outer">
        <h3>Tooltip with Vue3 Teleport</h3>
        <button id="show-modal" @click="show">显示</button>
        <Teleport to="body">
            <modal :show="showModal" @close="hide">
                <template #header>
                    <h3>custom modal</h3>
                </template>
            </modal>
        </Teleport>
    </div>
</template>

<script setup>
   import Modal from './Modal.vue'
   import {ref} from 'vue'

   const showModal = ref(false)
   const show = ()=>{
       showModal.value = true
   }
   const hide=()=>{
       showModal.value = false
   }

</script>

// Modal.vue
<template>
<Transition name="modal">
    <div v-if="show" class="modal-mask">
       <div class="modal-wrapper">
           <div class="modal-container">
               <div class="modal-header">
               <slot name="header">default header</slot>
               </div>
               <div class="modal-body">
                   <slot name="body">default body</slot>
               </div>
               <div class="modal-footer">
                   <slot name="footer">default footer</slot>
                   <button class="modal-default-button" @click="$emit('close')">x</button>
               </div>
           </div>
       </div>
    </div>
</Transition>

</template>

<script setup>
    import {defineProps} from 'vue'
    const props=defineProps({
        show:Boolean
    })
</script>

<style scoped>
    .modal-mask {
        position: fixed;
        z-index: 9998;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        display: table;
        transition: opacity 0.3s ease;
    }

    .modal-wrapper {
        display: table-cell;
        vertical-align: middle;
    }

    .modal-container {
        width: 300px;
        margin: 0 auto;
        padding: 20px 30px;
        background-color: #fff;
        border-radius: 2px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
        transition: all 0.3s ease;
    }

    .modal-header h3 {
        margin-top: 0;
        color: #42b983;
    }

    .modal-body {
        margin: 20px 0;
    }

    .modal-default-button {
        float: right;
    }
    .modal-enter-from {
        opacity: 0;
    }

    .modal-leave-to {
        opacity: 0;
    }

    .modal-enter-from .modal-container,
    .modal-leave-to .modal-container {
        -webkit-transform: scale(1.1);
        transform: scale(1.1);
    }
</style>
复制代码
```

注意，Teleport的to目标必须已经存在于DOM中，在挂载Teleport前，to目标的元素必须挂载完成，否则会报错

### 实现原理

主要是通过**document的querySelector**实现的，Teleport实质是一个组件，必然会经历组件的初始化、挂载与更新

1. Teleport组件通过process初始化，查看是否禁用teleport，禁用则在初始化容器时渲染，处理HMR
2. 通过resolveTarget拿到要渲染teleport的容器节点，目标不存在就报错返回，存在打注释定位标记
3. 进入挂载阶段，确认内部是否是array children结构，是否配置了disabled参数，配置了就解析在默认容器中，没有配置就解析在to 指定的容器中，初始化结束
4. 更新阶段，核心函数moveTeleport，能触发Teleport更新只有两种情况:

- 修改to或者disabled的值
- 组件内部的内容发生更新

根据这两种情况进行不同的处理，更新结束

**moveTeleport** 用于更新

修改to或者disabled的值分为以下情况:

1. disabled从false为true，会将to指定的容器移动到默认的容器
2. disabled从true为false，会将默认容器移动到to指定的容器
3. to指定的容器改变，向重新传递一个选择器，重新解析，移动到新的选择器位置

```js
function moveTeleport(
  vnode: VNode,
  container: RendererElement,
  parentAnchor: RendererNode | null,
  { o: { insert }, m: move }: RendererInternals,
  moveType: TeleportMoveTypes = TeleportMoveTypes.REORDER
) {
  // 确认新的目标容器，目标容器变了，将Teleport插入新的目标容器
  if (moveType === TeleportMoveTypes.TARGET_CHANGE) {
    insert(vnode.targetAnchor!, container, parentAnchor)
  }
  const { el, anchor, shapeFlag, children, props } = vnode
  const isReorder = moveType === TeleportMoveTypes.REORDER
  // 重新排序
  if (isReorder) {
    insert(el!, container, parentAnchor)
  }
  // 没有重排序或者disabled属性设置为true
  if (!isReorder || isTeleportDisabled(props)) {
    // 遍历vnode子节点，将子节点移动到
    if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
      for (let i = 0; i < (children as VNode[]).length; i++) {
        move(
          (children as VNode[])[i],
          container,
          parentAnchor,
          MoveType.REORDER
        )
      }
    }
  }
  // 需要重新排序
  if (isReorder) {
    insert(anchor!, container, parentAnchor)
  }
}
复制代码
```

## nextTick实现原理

以下代码点击按钮以后输出是什么

```js
<template>
  <div class="container">
    <div ref="test">{{msg}}</div>
    <button @click="handleClick">点击</button>
  </div>
</template>
<script>
  export default {
    data(){
      return{
        msg:'Hello,lyllovelemon'
      }
    },
    methods:{
      handleClick(){
        this.msg='Hello,lyl'
        console.log(this.$refs.test.innerHTML)
      }
    }
  }
</script>	
复制代码
```

输出结果是Hello,lyllovelemon，这是为什么呢

### 异步更新DOM策略

Vue的DOM更新是异步的，当一个响应式数据变化时，会在Watcher的setter函数中通知闭包中的Dep，Dep会调用它管理的所有Watcher对象，触发update方法

```js
//vue2.7.10 src/core/observer/watcher.ts
update() {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    this.run() // 同步执行run方法，直接渲染视图
  } else {
    queueWatcher(this) // 异步执行queueWatcher，加到观察者队列中，在下一个tick调用
  }
}
复制代码
```

我们再看看queueWatcher的具体实现

```js
// src/core/observer/scheduler.ts
// 接收一个Watcher实例作为参数
export function queueWatcher(watcher: Watcher) {
  const id = watcher.id // 获取watcher的id
  if (has[id] != null) { // 观察者队列中存在就跳过
    return
  }
  
  if (watcher === Dep.target && watcher.noRecurse) { // noRecurse为true
    return
  }

  has[id] = true // 加入到has哈希表，用于下次校验
  if (!flushing) {
    queue.push(watcher) // 没有flush到加到队列
  } else {
    // flush过了, 根据id从watcher队列中删除
    // if already past its id, it will be run next immediately.
    let i = queue.length - 1
    while (i > index && queue[i].id > watcher.id) {
      i--
    }
    queue.splice(i + 1, 0, watcher)
  }
  // queue the flush
  if (!waiting) {
    waiting = true // waiting标志为true，表示不是立即更新视图，而是进入等待

    if (__DEV__ && !config.async) {
      flushSchedulerQueue() // 开发环境且同步
      return
    }
    // 在nextTick中执行flushSchedulerQueue方法
    nextTick(flushSchedulerQueue)
  }
}

复制代码
```

queueWatcher中可以看到视图更新不是立即进行，而是将watcher对象push到一个队列，此时处于waiting状态，watch对象会不停的加入到队列中，等到下一个tick时，这些对象才会被遍历取出，更新视图。下一个tick就是nextTick

**Vue为什么不使用同步更新DOM**

```js
<template>
  <div class="container">
    <div>{{count}}</div>
    <button @click="handleClick">点击</button>
  </div>
</template>
<script>
  export default {
    data(){
      return{
        count:0
      }
    },
    methods:{
      handleClick(){
        for(let i=0;i<1000;i++){
           this.count++
        }
      }
    }
  }
</script>
复制代码
```

如上图，点击按钮时，执行了1000次 count的++，当DOM更新是同步时，就会触发1000次DOM更新刷新视图，这是很耗性能的，Vue使用异步更新DOM策略，将count++的操作放到队列中，在下一个tick时统一执行。同一个id的Watcher不会重复加到queue中，所以最终更新视图直接将count从0加到1000，保证视图更新DOM是从当前栈执行完后的下一个tick调用，大大优化了性能

### nextTick原理

vue2.5以前的版本：nextTick实质是产生一个回调函数加入到task（宏任务）或者microtask(微任务)，当前栈执行完后调用该回调函数，起到了异步触发的作用

vue2.5以后全部使用微任务实现，原因是使用宏任务会产生一些问题

```js
 // src/core/util/next-tick.ts
export let isUsingMicroTask = false

const callbacks: Array<Function> = []
let pending = false
function flushCallbacks() { //清空回调函数
  pending = false  // pending为false，表示等待结束，准备执行
  const copies = callbacks.slice(0) // 浅拷贝回调函数
  callbacks.length = 0 // 将回调函数清空
  for (let i = 0; i < copies.length; i++) { // 拷贝后的数组遍历执行回调
    copies[i]()
  }
}

let timerFunc // 延迟执行函数

/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  // 支持promise，用promise实现
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
 
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (
  !isIE &&
  typeof MutationObserver !== 'undefined' &&
  (isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]')
) {
  // 不支持promise但是支持MutationObserver，用MutationObserver实现
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // 以上不支持但支持setImmediate，用setImmediate实现
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // 以上都不支持，最后用setTimeout实现
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
export function nextTick(cb?: (...args: any[]) => any, ctx?: object) {
  let _resolve
  // cb存到callbacks中
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e: any) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true // pending是一个状态标志，保证timerFunc在下一个tick之前只执行一次
    timerFunc()//timerFunc
  }
  // $flow-disable-line
  // 支持promise就用promise实现
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}   
复制代码
```

timerFunc函数做了什么：先后按照promise、MutationObserver、setImmediate、setTimeout实现，promise、MutationObserver都是微任务的实现

**为什么优先使用微任务**

由于浏览器的事件循环机制，引擎在每一个宏任务执行完毕，从队列中取下一个宏任务执行之前，会将这个宏任务下的微任务队列拿出来依次执行，因此微任务的执行时间早于宏任务。每个task执行完后都会触发UI的重新渲染，在microTask中完成数据更新，当前task结束就能拿到最新的UI了，如果再新建一个task，UI渲染就会进行两次

**为什么优先使用Promise而不是MutationObserver**

MutationObserver虽然浏览器兼容性更好，但是在iOS7，Android 4.4的touch事件上会有问题

**setImmediate为什么比setTimeout好**

setImmediate可以保证调用后立即执行，setTimeout需要和系统时间保持一致，最快也要4ms以后才能执行。但是setTimeout的浏览器兼容性更好，setImmediate只支持IE浏览器

## watch函数实现原理

### watch原理

对watch的每一个属性创建watcher，watcher在初始化时会将监听的目标值缓存到watcher.value中，触发data[key]的get方法，被对应的dep进行依赖收集，当data[key]发生改变时触发set方法，执行dep.notify方法，

通知所有收集的依赖，触发收集的watcher的watch，执行watch.cb,也就是watch中的监听函数

### computed原理

**computed是响应式的,** 给computed设置get和set会和Object.defineProperty关联起来(vue2),vue3会和proxy关联

**computed是有缓存的**，主要通过dirty控制

dirty是一个脏数据标志位，为false时表示读取computed使用缓存，为true时表示读取computed会执行get函数重新计算

```js
    export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>,
  debugOptions?: DebuggerOptions
) {
  let getter: ComputedGetter<T> // 收集getter
  let setter: ComputedSetter<T> // 收集setter

  const onlyGetter = isFunction(getterOrOptions)
  if (onlyGetter) {
    getter = getterOrOptions
    setter = __DEV__
      ? () => {
        warn('Write operation failed: computed value is readonly')
      }
      : noop
  } else {
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }

  // 是服务端渲染就赋值null，客户端渲染实例化watcher
  const watcher = isServerRendering()
    ? null
    : new Watcher(currentInstance, getter, noop, { lazy: true })

  if (__DEV__ && watcher && debugOptions) { // 用于debug
    watcher.onTrack = debugOptions.onTrack
    watcher.onTrigger = debugOptions.onTrigger
  }

  const ref = {
    // some libs rely on the presence effect for checking computed refs
    // from normal refs, but the implementation doesn't matter
    effect: watcher,
    get value() {
      if (watcher) {
        if (watcher.dirty) { // dirty标志位，表示数据需要更新
          watcher.evaluate() // 执行watcher的evaluate方法
        }
        if (Dep.target) {
          if (__DEV__ && Dep.target.onTrack) {
            Dep.target.onTrack({
              effect: Dep.target,
              target: ref,
              type: TrackOpTypes.GET,
              key: 'value'
            })
          }
          watcher.depend()
        }
        return watcher.value
      } else {
        return getter()
      }
    },
    set value(newVal) {
      setter(newVal)
    }
  } as any

  def(ref, RefFlag, true)
  def(ref, ReactiveFlags.IS_READONLY, onlyGetter)

  return ref
}
复制代码
```

## 性能优化

[cn.vuejs.org/guide/best-…](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fbest-practices%2Fperformance.html)

性能优化主要在以下几个方面:**页面加载速度优化**，**打包体积优化，打包速度优化,浏览器安全**

### 页面加载速度优化

1. 使用正确的架构

**SPA/SSR/SSG**

1. v-for循环中正确的使用key

key需要在循环列表中保持唯一，不要使用数组下标作为key，key用于高效的更新diff

1. 保持props稳定

如下图代码，它使用了id和activeId两个prop来保证它是否是当前活跃的一项，代码存在什么问题呢？

```ini
<ListItem
  v-for="item in list"
  :id="item.id"
  :active-id="activeId" />
复制代码
```

当activeId更新时，ListItem的每一项都会进行更新，造成了不必要的重复渲染，我们需要改成只有活跃状态发生改变的项才需要更新,将上面的代码改写。对于大多数的组件来说，activeId 改变时，它们的 active prop 都会保持不变，因此它们无需再更新。总结一下，就是让传给子组件的 props 尽量保持稳定。

```ini
<ListItem
  v-for="item in list"
  :id="item.id"
  :active="item.id === activeId" />
复制代码
```

1. 正确使用[v-once](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fapi%2Fbuilt-in-directives.html%23v-once)和[v-memo](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fapi%2Fbuilt-in-directives.html%23v-memo)
2. 使用shallowRef和shallowReactive绕开深度响应

Vue3的响应式默认是深度的，优点在于便于进行数据的状态管理，缺点是数据量大时，性能负担加重，因为每个属性访问都会进行依赖的深度追踪。

解决办法：使用shallowRef和shallowReactive绕开深度响应

```scss
const shallowArray = shallowRef([
  /* 巨大的列表，里面包含深层的对象 */
])

// 这不会触发更新...
shallowArray.value.push(newObject)
// 这才会触发更新
shallowArray.value = [...shallowArray.value, newObject]

// 这不会触发更新...
shallowArray.value[0].foo = 1
// 这才会触发更新
shallowArray.value = [  {    ...shallowArray.value[0],
    foo: 1
  },
  ...shallowArray.value.slice(1)
]
复制代码
```

1. 长列表渲染：虚拟列表

长列表渲染的最优方案是虚拟列表，其次还有time slice和css方案。渲染的列表项成千上万时，对应的DOM元素也会递增，但是我们并不需要一次性将所有数据渲染出来，虚拟列表的原理就是创建一个可视化区域，让可视化区域的数据渲染出来

1. 避免内存泄漏

事件绑定后需要取消绑定，定时器也需要定时销毁

```javascript
created(){
  window.addEventListener('click',this.onClick,false)
  setTimeout(timer,2000)
}
beforeUnmount(){
  window.removeEventListener('click',this.onClick,false)
  clearTimeout(timer,2000)
}  
复制代码
```

1. 预加载与懒加载

图片，路由都可以进行懒加载

1. v-if和v-show使用
2. computed和watch使用
3. 使用keep-alive缓存DOM
4. 节流/防抖
5. 浏览器缓存与本地缓存
6. CSS优化

**GPU加速**

**减少回流和重绘**

### 打包速度优化

1. 开启多进程打包
2. 多线程打包
3. CDN

### 打包体积优化

以打包工具webpack举例

1. 开启gzip压缩
2. 压缩HTML
3. 压缩CSS
4. 压缩JavaScript
5. 图片压缩
6. 尽量选择ES模式的依赖

选择lodash-es优于lodash，它对于tree-shaking更友好

1. 使用Html-webpack-bundle-analyaser分析包体积

这个插件可用于分析包体积，查看最大的包，按需进行体积优化

1. 代码分割

使用打包工具将整个包拆分为多个较小的包，可以按需加载或者并行加载，通过适当的代码分割，页面加载时需要的功能可以立即下载，而额外的块只在需要时才加载，从而提高性能。

```javascript
// defineAsyncComponent可用于按需加载组件
import {defineAsyncComponent} from "vue"
// Container是一个异步组件，按需加载    
const Container=defineAsyncComponent(()=>import("./container.vue"))  
复制代码
```

1. 开启tree-shaking
2. 第三方库按需引入

以element为例，项目中并没有用到所有的组件时，可以进行按需引入，可以有效减少第三方包体积

1. 使用合适的sourceMap

开发环境最优sourceMap：cheap-module-eval-source-map

生产环境推荐souceMap:source-map

### 浏览器安全

1. 避免CSRF
2. 避免XSS
3. 安全沙箱

## 写在最后

源码分析花的时间很长，白天要工作（目前工作比较饱和），只能晚上写，前前后后花了快1个月时间，如果这篇文章对你有帮助的话，不妨给博主姐姐点赞收藏评论



作者：lyllovelemon
链接：https://juejin.cn/post/7195517440344211512
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。