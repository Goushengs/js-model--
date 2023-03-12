### 前言

时间过得太快 一晃半年都没写文章了 哈哈 如今又到了一年一度的**金三银四**的好时节 我李汉三回来了

先说一下写此系列的**初衷**--最近公司招聘高级前端岗位 于是面了很多人 这面着面着就有点 *痛心疾首* 了 因为我发现很多做了五年以上的同学还是停留在业务逻辑层 一般表现为项目经验丰富 做过一些复杂的业务 但是对于框架或者工具的底层源码实现以及 js 的细节掌握并不到位 就比如 Vue 的混入（mixin） 大家基本都知道用它能解决啥问题 但是混入的原理 混入策略 混入顺序等等这些能全部回答出来的就比较少 于是我决定和大家一起手写一遍**Vue2.0**版本的源码 学习优秀源码的思路（后续会加入 3.0 版本）

**适用人群：** 没时间去看官方源码或者看源码看的比较懵而不想去看的同学

**注意**： 我看源码一直保持 28 策略 所谓百分之 20 的代码实现了百分之 80 的功能 所以此系列咱们只关心核心逻辑以及功能的实现 **不包含** 打包构建 类型检查 跨平台 报错提醒 边界处理 兼容处理 SSR（服务端渲染）等

> 强烈建议 大家有时间还是把核心代码掌握之后再回头去看一遍官方源码

------

### 正文

大家都知道 Vue 的一个核心特点是**数据驱动** 如果按照以往 Jquery 的思想 咱们数据变化了想要同步到视图就必须要手动操作 dom 更新 但是 Vue 帮我们做到了数据变动自动更新视图的功能 那在 Vue 内部就一定有一个机制能监听到数据变化然后触发更新 本篇主要介绍**响应式数据**的原理

#### 1.数据初始化

```javascript
new Vue({
  el: "#app",
  router,
  store,
  render: (h) => h(App),
});
复制代码
```

这段代码 大家一定非常熟悉 这就是 Vue 实例化的过程 从 new 操作符 咱们可以看出 Vue 其实就是一个构造函数 没啥特别的 传入的参数就是一个对象 我们叫做 options（选项）

```javascript
// src/index.js

import { initMixin } from "./init.js";

// Vue就是一个构造函数 通过new关键字进行实例化
function Vue(options) {
  // 这里开始进行Vue初始化工作
  this._init(options);
}
// _init方法是挂载在Vue原型的方法 通过引入文件的方式进行原型挂载需要传入Vue
// 此做法有利于代码分割
initMixin(Vue);
export default Vue;
复制代码
```

因为在 Vue 初始化可能会处理很多事情 比如数据处理 事件处理 生命周期处理等等 所以划分不同文件引入利于代码分割

```javascript
// src/init.js
import { initState } from "./state";
export function initMixin(Vue) {
  Vue.prototype._init = function (options) {
    const vm = this;
    // 这里的this代表调用_init方法的对象(实例对象)
    //  this.$options就是用户new Vue的时候传入的属性
    vm.$options = options;
    // 初始化状态
    initState(vm);
  };
}
复制代码
```

initMixin 把_init 方法挂载在 Vue 原型 供 Vue 实例调用

```javascript
// src/state.js
import { observe } from "./observer/index.js";

// 初始化状态 注意这里的顺序 比如我经常面试会问到 是否能在data里面直接使用prop的值 为什么？
// 这里初始化的顺序依次是 prop>methods>data>computed>watch
export function initState(vm) {
  // 获取传入的数据对象
  const opts = vm.$options;
  if (opts.props) {
    initProps(vm);
  }
  if (opts.methods) {
    initMethod(vm);
  }
  if (opts.data) {
    // 初始化data
    initData(vm);
  }
  if (opts.computed) {
    initComputed(vm);
  }
  if (opts.watch) {
    initWatch(vm);
  }
}

// 初始化data数据
function initData(vm) {
  let data = vm.$options.data;
  //   实例的_data属性就是传入的data
  // vue组件data推荐使用函数 防止数据在组件之间共享
  data = vm._data = typeof data === "function" ? data.call(vm) : data || {};

  // 把data数据代理到vm 也就是Vue实例上面 我们可以使用this.a来访问this._data.a
  for (let key in data) {
    proxy(vm, `_data`, key);
  }
  // 对数据进行观测 --响应式数据核心
  observe(data);
}
// 数据代理
function proxy(object, sourceKey, key) {
  Object.defineProperty(object, key, {
    get() {
      return object[sourceKey][key];
    },
    set(newValue) {
      object[sourceKey][key] = newValue;
    },
  });
}
复制代码
```

initState 咱们主要关注 initData 里面的 observe 是响应式数据核心 所以另建 observer 文件夹来专注响应式逻辑 其次我们还做了一层数据代理 把data代理到实例对象this上

#### 2.对象的数据劫持

```javascript
// src/obserber/index.js
class Observer {
  // 观测值
  constructor(value) {
    this.walk(value);
  }
  walk(data) {
    // 对象上的所有属性依次进行观测
    let keys = Object.keys(data);
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i];
      let value = data[key];
      defineReactive(data, key, value);
    }
  }
}
// Object.defineProperty数据劫持核心 兼容性在ie9以及以上
function defineReactive(data, key, value) {
  observe(value); // 递归关键
  // --如果value还是一个对象会继续走一遍odefineReactive 层层遍历一直到value不是对象才停止
  //   思考？如果Vue数据嵌套层级过深 >>性能会受影响
  Object.defineProperty(data, key, {
    get() {
      console.log("获取值");
      return value;
    },
    set(newValue) {
      if (newValue === value) return;
      console.log("设置值");
      value = newValue;
    },
  });
}
export function observe(value) {
  // 如果传过来的是对象或者数组 进行属性劫持
  if (
    Object.prototype.toString.call(value) === "[object Object]" ||
    Array.isArray(value)
  ) {
    return new Observer(value);
  }
}
复制代码
```

数据劫持核心是 defineReactive 函数 主要使用 Object.defineProperty 来对数据 get 和 set 进行劫持 这里就解决了之前的问题 为啥数据变动了会自动更新视图 我们可以在 set 里面去通知视图更新

> 思考 1.这样的数据劫持方式对数组有什么影响？

*这样递归的方式其实无论是对象还是数组都进行了观测 但是我们想一下此时如果 data 包含数组比如 a:[1,2,3,4,5] 那么我们根据下标可以直接修改数据也能触发 set 但是如果一个数组里面有上千上万个元素 每一个元素下标都添加 get 和 set 方法 这样对于性能来说是承担不起的 所以此方法只用来劫持对象*

> 思考 2.Object.defineProperty 缺点？

*对象新增或者删除的属性无法被 set 监听到 只有对象本身存在的属性修改才会被劫持*

#### 3.数组的观测

```javascript
// src/obserber/index.js
import { arrayMethods } from "./array";
class Observer {
  constructor(value) {
    if (Array.isArray(value)) {
      // 这里对数组做了额外判断
      // 通过重写数组原型方法来对数组的七种方法进行拦截
      value.__proto__ = arrayMethods;
      // 如果数组里面还包含数组 需要递归判断
      this.observeArray(value);
    } else {
      this.walk(value);
    }
  }
  observeArray(items) {
    for (let i = 0; i < items.length; i++) {
      observe(items[i]);
    }
  }
}
复制代码
```

因为对数组下标的拦截太浪费性能 对 Observer 构造函数传入的数据参数增加了数组的判断

```javascript
// src/obserber/index.js
class Observer {
  // 观测值
  constructor(value) {
    Object.defineProperty(value, "__ob__", {
      //  值指代的就是Observer的实例
      value: this,
      //  不可枚举
      enumerable: false,
      writable: true,
      configurable: true,
    });
  }
}
复制代码
```

对数组原型重写之前咱们先要理解这段代码 这段代码的意思就是给每个响应式数据增加了一个不可枚举的__ob__属性 并且指向了 Observer 实例 那么我们首先可以根据这个属性来防止已经被响应式观察的数据反复被观测 其次 响应式数据可以使用__ob__来获取 Observer 实例的相关方法 这对数组很关键

```javascript
// src/obserber/array.js
// 先保留数组原型
const arrayProto = Array.prototype;
// 然后将arrayMethods继承自数组原型
// 这里是面向切片编程思想（AOP）--不破坏封装的前提下，动态的扩展功能
export const arrayMethods = Object.create(arrayProto);
let methodsToPatch = [
  "push",
  "pop",
  "shift",
  "unshift",
  "splice",
  "reverse",
  "sort",
];
methodsToPatch.forEach((method) => {
  arrayMethods[method] = function (...args) {
    //   这里保留原型方法的执行结果
    const result = arrayProto[method].apply(this, args);
    // 这句话是关键
    // this代表的就是数据本身 比如数据是{a:[1,2,3]} 那么我们使用a.push(4)  this就是a  ob就是a.__ob__ 这个属性就是上段代码增加的 代表的是该数据已经被响应式观察过了指向Observer实例
    const ob = this.__ob__;

    // 这里的标志就是代表数组有新增操作
    let inserted;
    switch (method) {
      case "push":
      case "unshift":
        inserted = args;
        break;
      case "splice":
        inserted = args.slice(2);
      default:
        break;
    }
    // 如果有新增的元素 inserted是一个数组 调用Observer实例的observeArray对数组每一项进行观测
    if (inserted) ob.observeArray(inserted);
    // 之后咱们还可以在这里检测到数组改变了之后从而触发视图更新的操作--后续源码会揭晓
    return result;
  };
});
复制代码
```

#### 4.响应式数据的思维导图

![图片](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96313216e0694b589b4584c9e109bb76~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此Vue的响应式数据原理已经完结 大家可以看着思维导图自己动手写一遍核心代码哈 需要注意的是 里面对于this的引用很多 不同的环境this的指向不同 大家不要搞混淆  然后目前能实现的功能仅仅是对数据进行了响应式观测 但是对数据修改后怎么导致视图重新渲染 这块还需要结合Watcher和dep 采用观察者模式实现依赖收集和派发更新的过程

此篇主要手写 Vue2.0 源码-**模板编译原理**

上一篇咱们主要介绍了 Vue 数据的[响应式原理](https://juejin.cn/post/6935344605424517128) 对于中高级前端来说 响应式原理基本是面试 Vue 必考的源码基础类 如果不是很清楚的话基本就被 pass 了 那么今天咱们手写的模板编译原理也是 Vue 面试比较**频繁**的一个点 而且复杂程度是高于响应式原理的 里面主要涉及到 ast 以及大量正则匹配 大家学习完可以看着思维导图一起手写一遍加深印象哈

**适用人群：** 没时间去看官方源码或者看源码看的比较懵而不想去看的同学

> 建议： 想学习正则表达式的同学可以看看小编这篇文章 [前端进阶高薪必看-正则篇](https://juejin.cn/post/6844904153131515912)

------

### 正文

```javascript
// Vue实例化
new Vue({
  el: "#app",
  data() {
    return {
      a: 111,
    };
  },
  // render(h) {
  //   return h('div',{id:'a'},'hello')
  // },
  // template:`<div id="a">hello</div>`
});
复制代码
```

上面这段代码 大家一定不陌生 按照官网给出的生命周期图 咱们传入的 options 选项里面可以手动配置 template 或者是 render ![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/402a167b6c6040cf823cf04ab91cedec~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

> 注意一：平常开发中 我们使用的是不带编译版本的 Vue 版本（runtime-only）直接在 options 传入 template 选项 在开发环境报错

> 注意二：这里传入的 template 选项不要和.vue 文件里面的<template>模板搞混淆了 vue 单文件组件的 template 是需要 vue-loader 进行处理的

我们传入的 el 或者 template 选项最后都会被解析成 render 函数 这样才能保持模板解析的一致性

#### 1.模板编译入口

```javascript
// src/init.js

import { initState } from "./state";
import { compileToFunctions } from "./compiler/index";
export function initMixin(Vue) {
  Vue.prototype._init = function (options) {
    const vm = this;
    // 这里的this代表调用_init方法的对象(实例对象)
    //  this.$options就是用户new Vue的时候传入的属性
    vm.$options = options;
    // 初始化状态
    initState(vm);

    // 如果有el属性 进行模板渲染
    if (vm.$options.el) {
      vm.$mount(vm.$options.el);
    }
  };

  // 这块代码在源码里面的位置其实是放在entry-runtime-with-compiler.js里面
  // 代表的是Vue源码里面包含了compile编译功能 这个和runtime-only版本需要区分开
  Vue.prototype.$mount = function (el) {
    const vm = this;
    const options = vm.$options;
    el = document.querySelector(el);

    // 如果不存在render属性
    if (!options.render) {
      // 如果存在template属性
      let template = options.template;

      if (!template && el) {
        // 如果不存在render和template 但是存在el属性 直接将模板赋值到el所在的外层html结构（就是el本身 并不是父元素）
        template = el.outerHTML;
      }

      // 最终需要把tempalte模板转化成render函数
      if (template) {
        const render = compileToFunctions(template);
        options.render = render;
      }
    }
  };
}
复制代码
```

咱们主要关心$mount 方法 最终将处理好的 template 模板转成 render 函数

#### 2.模板转化核心方法 compileToFunctions

```javascript
// src/compiler/index.js

import { parse } from "./parse";
import { generate } from "./codegen";
export function compileToFunctions(template) {
  // 我们需要把html字符串变成render函数
  // 1.把html代码转成ast语法树  ast用来描述代码本身形成树结构 不仅可以描述html 也能描述css以及js语法
  // 很多库都运用到了ast 比如 webpack babel eslint等等
  let ast = parse(template);
  // 2.优化静态节点
  // 这个有兴趣的可以去看源码  不影响核心功能就不实现了
  //   if (options.optimize !== false) {
  //     optimize(ast, options);
  //   }

  // 3.通过ast 重新生成代码
  // 我们最后生成的代码需要和render函数一样
  // 类似_c('div',{id:"app"},_c('div',undefined,_v("hello"+_s(name)),_c('span',undefined,_v("world"))))
  // _c代表创建元素 _v代表创建文本 _s代表文Json.stringify--把对象解析成文本
  let code = generate(ast);
  //   使用with语法改变作用域为this  之后调用render函数可以使用call改变this 方便code里面的变量取值
  let renderFn = new Function(`with(this){return ${code}}`);
  return renderFn;
}
复制代码
```

新建 compiler 文件夹 表示编译相关功能 核心导出 compileToFunctions 函数 主要有三个步骤 1.生成 ast 2.优化静态节点 3.根据 ast 生成 render 函数

#### 3.解析 html 并生成 ast

```javascript
// src/compiler/parse.js

// 以下为源码的正则  对正则表达式不清楚的同学可以参考小编之前写的文章(前端进阶高薪必看 - 正则篇);
const ncname = `[a-zA-Z_][\\-\\.0-9_a-zA-Z]*`; //匹配标签名 形如 abc-123
const qnameCapture = `((?:${ncname}\\:)?${ncname})`; //匹配特殊标签 形如 abc:234 前面的abc:可有可无
const startTagOpen = new RegExp(`^<${qnameCapture}`); // 匹配标签开始 形如 <abc-123 捕获里面的标签名
const startTagClose = /^\s*(\/?)>/; // 匹配标签结束  >
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`); // 匹配标签结尾 如 </abc-123> 捕获里面的标签名
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/; // 匹配属性  形如 id="app"

let root, currentParent; //代表根节点 和当前父节点
// 栈结构 来表示开始和结束标签
let stack = [];
// 标识元素和文本type
const ELEMENT_TYPE = 1;
const TEXT_TYPE = 3;
// 生成ast方法
function createASTElement(tagName, attrs) {
  return {
    tag: tagName,
    type: ELEMENT_TYPE,
    children: [],
    attrs,
    parent: null,
  };
}

// 对开始标签进行处理
function handleStartTag({ tagName, attrs }) {
  let element = createASTElement(tagName, attrs);
  if (!root) {
    root = element;
  }
  currentParent = element;
  stack.push(element);
}

// 对结束标签进行处理
function handleEndTag(tagName) {
  // 栈结构 []
  // 比如 <div><span></span></div> 当遇到第一个结束标签</span>时 会匹配到栈顶<span>元素对应的ast 并取出来
  let element = stack.pop();
  // 当前父元素就是栈顶的上一个元素 在这里就类似div
  currentParent = stack[stack.length - 1];
  // 建立parent和children关系
  if (currentParent) {
    element.parent = currentParent;
    currentParent.children.push(element);
  }
}

// 对文本进行处理
function handleChars(text) {
  // 去掉空格
  text = text.replace(/\s/g, "");
  if (text) {
    currentParent.children.push({
      type: TEXT_TYPE,
      text,
    });
  }
}

// 解析标签生成ast核心
export function parse(html) {
  while (html) {
    // 查找<
    let textEnd = html.indexOf("<");
    // 如果<在第一个 那么证明接下来就是一个标签 不管是开始还是结束标签
    if (textEnd === 0) {
      // 如果开始标签解析有结果
      const startTagMatch = parseStartTag();
      if (startTagMatch) {
        // 把解析好的标签名和属性解析生成ast
        handleStartTag(startTagMatch);
        continue;
      }

      // 匹配结束标签</
      const endTagMatch = html.match(endTag);
      if (endTagMatch) {
        advance(endTagMatch[0].length);
        handleEndTag(endTagMatch[1]);
        continue;
      }
    }

    let text;
    // 形如 hello<div></div>
    if (textEnd >= 0) {
      // 获取文本
      text = html.substring(0, textEnd);
    }
    if (text) {
      advance(text.length);
      handleChars(text);
    }
  }

  // 匹配开始标签
  function parseStartTag() {
    const start = html.match(startTagOpen);

    if (start) {
      const match = {
        tagName: start[1],
        attrs: [],
      };
      //匹配到了开始标签 就截取掉
      advance(start[0].length);

      // 开始匹配属性
      // end代表结束符号>  如果不是匹配到了结束标签
      // attr 表示匹配的属性
      let end, attr;
      while (
        !(end = html.match(startTagClose)) &&
        (attr = html.match(attribute))
      ) {
        advance(attr[0].length);
        attr = {
          name: attr[1],
          value: attr[3] || attr[4] || attr[5], //这里是因为正则捕获支持双引号 单引号 和无引号的属性值
        };
        match.attrs.push(attr);
      }
      if (end) {
        //   代表一个标签匹配到结束的>了 代表开始标签解析完毕
        advance(1);
        return match;
      }
    }
  }
  //截取html字符串 每次匹配到了就往前继续匹配
  function advance(n) {
    html = html.substring(n);
  }
  //   返回生成的ast
  return root;
}
复制代码
```

利用正则 匹配 html 字符串 遇到开始标签 结束标签和文本 解析完毕之后生成对应的 ast 并建立相应的父子关联 不断的 advance 截取剩余的字符串 直到 html 全部解析完毕 咱们这里主要写了对于开始标签里面的属性的处理--parseStartTag

#### 4.根据 ast 重新生成代码

```javascript
// src/compiler/codegen.js

const defaultTagRE = /\{\{((?:.|\r?\n)+?)\}\}/g; //匹配花括号 {{  }} 捕获花括号里面的内容

function gen(node) {
  // 判断节点类型
  // 主要包含处理文本核心
  // 源码这块包含了复杂的处理  比如 v-once v-for v-if 自定义指令 slot等等  咱们这里只考虑普通文本和变量表达式{{}}的处理

  // 如果是元素类型
  if (node.type == 1) {
    //   递归创建
    return generate(node);
  } else {
    //   如果是文本节点
    let text = node.text;
    // 不存在花括号变量表达式
    if (!defaultTagRE.test(text)) {
      return `_v(${JSON.stringify(text)})`;
    }
    // 正则是全局模式 每次需要重置正则的lastIndex属性  不然会引发匹配bug
    let lastIndex = (defaultTagRE.lastIndex = 0);
    let tokens = [];
    let match, index;

    while ((match = defaultTagRE.exec(text))) {
      // index代表匹配到的位置
      index = match.index;
      if (index > lastIndex) {
        //   匹配到的{{位置  在tokens里面放入普通文本
        tokens.push(JSON.stringify(text.slice(lastIndex, index)));
      }
      //   放入捕获到的变量内容
      tokens.push(`_s(${match[1].trim()})`);
      //   匹配指针后移
      lastIndex = index + match[0].length;
    }
    // 如果匹配完了花括号  text里面还有剩余的普通文本 那么继续push
    if (lastIndex < text.length) {
      tokens.push(JSON.stringify(text.slice(lastIndex)));
    }
    // _v表示创建文本
    return `_v(${tokens.join("+")})`;
  }
}

// 处理attrs属性
function genProps(attrs) {
  let str = "";
  for (let i = 0; i < attrs.length; i++) {
    let attr = attrs[i];
    // 对attrs属性里面的style做特殊处理
    if (attr.name === "style") {
      let obj = {};
      attr.value.split(";").forEach((item) => {
        let [key, value] = item.split(":");
        obj[key] = value;
      });
      attr.value = obj;
    }
    str += `${attr.name}:${JSON.stringify(attr.value)},`;
  }
  return `{${str.slice(0, -1)}}`;
}

// 生成子节点 调用gen函数进行递归创建
function getChildren(el) {
  const children = el.children;
  if (children) {
    return `${children.map((c) => gen(c)).join(",")}`;
  }
}
// 递归创建生成code
export function generate(el) {
  let children = getChildren(el);
  let code = `_c('${el.tag}',${
    el.attrs.length ? `${genProps(el.attrs)}` : "undefined"
  }${children ? `,${children}` : ""})`;
  return code;
}
复制代码
```

拿到生成好的 ast 之后 需要把 ast 转化成类似_c('div',{id:"app"},_c('div',undefined,_v("hello"+_s(name)),_c('span',undefined,_v("world"))))这样的字符串

#### 5.code 字符串生成 render 函数

```javascript
export function compileToFunctions(template) {
  let code = generate(ast);
  // 使用with语法改变作用域为this  之后调用render函数可以使用call改变this 方便code里面的变量取值 比如 name值就变成了this.name
  let renderFn = new Function(`with(this){return ${code}}`);
  return renderFn;
}
复制代码
```

#### 6.模板编译的思维导图

![模板编译](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e7df45741974fdf918300d53bd1ebf8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结



 至此 Vue 的模板编译原理已经完结 大家可以看着思维导图自己动手写一遍核心代码哈 需要注意的是 本篇大量使用字符串拼接以及正则相关的知识 遇到不懂的地方可以多查阅资料 也欢迎评论留言 

### 前言

今天是个特别的日子  祝各位女神**女神节快乐**哈 封面我就放一张杀殿的帅照表达我的祝福 哈哈

此篇主要手写 Vue2.0 源码-**初始渲染原理**

上一篇咱们主要介绍了 Vue [模板编译原理](https://juejin.cn/post/6936024530016010276) 它是 Vue 生成虚拟 dom 的基础 模板编译最后转化成了 render 函数 之后又如何能生成真实的 dom 节点去替换掉 el 选项配置呢 那么通过此篇的学习就可以知道 Vue 初始渲染的流程 此篇主要包含虚拟 dom 以及真实 dom 的生成

**适用人群：** 没时间去看官方源码或者看源码看的比较懵而不想去看的同学

------

### 正文

#### 1.组件挂载入口

```javascript
// src/init.js

Vue.prototype.$mount = function (el) {
  const vm = this;
  const options = vm.$options;
  el = document.querySelector(el);

  // 如果不存在render属性
  if (!options.render) {
    // 如果存在template属性
    let template = options.template;

    if (!template && el) {
      // 如果不存在render和template 但是存在el属性 直接将模板赋值到el所在的外层html结构（就是el本身 并不是父元素）
      template = el.outerHTML;
    }

    // 最终需要把tempalte模板转化成render函数
    if (template) {
      const render = compileToFunctions(template);
      options.render = render;
    }
  }

  // 将当前组件实例挂载到真实的el节点上面
  return mountComponent(vm, el);
};
复制代码
```

接着看$mount 方法 我们主要关注最后一句话 mountComponent 就是组件实例挂载的入口函数 这个方法放在源码的 lifecycle 文件里面 代表了与生命周期相关 因为我们组件初始渲染前后对应有 beforeMount 和 mounted 生命周期钩子

#### 2.组件挂载核心方法 mountComponent

```javascript
// src/lifecycle.js
export function mountComponent(vm, el) {
  // 上一步模板编译解析生成了render函数
  // 下一步就是执行vm._render()方法 调用生成的render函数 生成虚拟dom
  // 最后使用vm._update()方法把虚拟dom渲染到页面

  // 真实的el选项赋值给实例的$el属性 为之后虚拟dom产生的新的dom替换老的dom做铺垫
  vm.$el = el;
  //   _update和._render方法都是挂载在Vue原型的方法  类似_init
  vm._update(vm._render());
}
复制代码
```

新建 lifecycle.js 文件 表示生命周期相关功能 核心导出 mountComponent 函数 主要使用 vm._update(vm._render())方法进行实例挂载

#### 3.render 函数转化成虚拟 dom 核心方法 _render

```javascript
// src/render.js

import { createElement, createTextNode } from "./vdom/index";

export function renderMixin(Vue) {
  Vue.prototype._render = function () {
    const vm = this;
    // 获取模板编译生成的render方法
    const { render } = vm.$options;
    // 生成vnode--虚拟dom
    const vnode = render.call(vm);
    return vnode;
  };

  // render函数里面有_c _v _s方法需要定义
  Vue.prototype._c = function (...args) {
    // 创建虚拟dom元素
    return createElement(...args);
  };

  Vue.prototype._v = function (text) {
    // 创建虚拟dom文本
    return createTextNode(text);
  };
  Vue.prototype._s = function (val) {
    // 如果模板里面的是一个对象  需要JSON.stringify
    return val == null
      ? ""
      : typeof val === "object"
      ? JSON.stringify(val)
      : val;
  };
}
复制代码
```

主要在原型定义了_render 方法 然后执行了 render 函数 我们知道模板编译出来的 render 函数核心代码主要 return 了 类似于_c('div',{id:"app"},_c('div',undefined,_v("hello"+_s(name)),_c('span',undefined,_v("world"))))这样的代码 那么我们还需要定义一下_c _v _s 这些函数才能最终转化成为虚拟 dom

```javascript
// src/vdom/index.js

// 定义Vnode类
export default class Vnode {
  constructor(tag, data, key, children, text) {
    this.tag = tag;
    this.data = data;
    this.key = key;
    this.children = children;
    this.text = text;
  }
}

// 创建元素vnode 等于render函数里面的 h=>h(App)
export function createElement(tag, data = {}, ...children) {
  let key = data.key;
  return new Vnode(tag, data, key, children);
}

// 创建文本vnode
export function createTextNode(text) {
  return new Vnode(undefined, undefined, undefined, undefined, text);
}
复制代码
```

新建 vdom 文件夹 代表虚拟 dom 相关功能 定义 Vnode 类 以及 createElement 和 createTextNode 方法最后都返回 vnode

#### 4.虚拟 dom 转化成真实 dom 核心方法 _update

```javascript
// src/lifecycle.js

import { patch } from "./vdom/patch";
export function lifecycleMixin(Vue) {
  // 把_update挂载在Vue的原型
  Vue.prototype._update = function (vnode) {
    const vm = this;
    // patch是渲染vnode为真实dom核心
    patch(vm.$el, vnode);
  };
}
复制代码
// src/vdom/patch.js

// patch用来渲染和更新视图 今天只介绍初次渲染的逻辑
export function patch(oldVnode, vnode) {
  // 判断传入的oldVnode是否是一个真实元素
  // 这里很关键  初次渲染 传入的vm.$el就是咱们传入的el选项  所以是真实dom
  // 如果不是初始渲染而是视图更新的时候  vm.$el就被替换成了更新之前的老的虚拟dom
  const isRealElement = oldVnode.nodeType;
  if (isRealElement) {
    // 这里是初次渲染的逻辑
    const oldElm = oldVnode;
    const parentElm = oldElm.parentNode;
    // 将虚拟dom转化成真实dom节点
    let el = createElm(vnode);
    // 插入到 老的el节点下一个节点的前面 就相当于插入到老的el节点的后面
    // 这里不直接使用父元素appendChild是为了不破坏替换的位置
    parentElm.insertBefore(el, oldElm.nextSibling);
    // 删除老的el节点
    parentElm.removeChild(oldVnode);
    return el;
  }
}
// 虚拟dom转成真实dom 就是调用原生方法生成dom树
function createElm(vnode) {
  let { tag, data, key, children, text } = vnode;
  //   判断虚拟dom 是元素节点还是文本节点
  if (typeof tag === "string") {
    //   虚拟dom的el属性指向真实dom
    vnode.el = document.createElement(tag);
    // 解析虚拟dom属性
    updateProperties(vnode);
    // 如果有子节点就递归插入到父节点里面
    children.forEach((child) => {
      return vnode.el.appendChild(createElm(child));
    });
  } else {
    //   文本节点
    vnode.el = document.createTextNode(text);
  }
  return vnode.el;
}

// 解析vnode的data属性 映射到真实dom上
function updateProperties(vnode) {
  let newProps = vnode.data || {};
  let el = vnode.el; //真实节点
  for (let key in newProps) {
    // style需要特殊处理下
    if (key === "style") {
      for (let styleName in newProps.style) {
        el.style[styleName] = newProps.style[styleName];
      }
    } else if (key === "class") {
      el.className = newProps.class;
    } else {
      // 给这个元素添加属性 值就是对应的值
      el.setAttribute(key, newProps[key]);
    }
  }
}
复制代码
```

_update 核心方法就是 patch 初始渲染和后续更新都是共用这一个方法 只是传入的第一个参数不同 初始渲染总体思路就是根据虚拟 dom(vnode) 调用原生 js 方法创建真实 dom 节点并替换掉 el 选项的位置

#### 5._render 和_update 原型方法的混入

```javascript
// src/index.js

import { initMixin } from "./init.js";
import { lifecycleMixin } from "./lifecycle";
import { renderMixin } from "./render";
// Vue就是一个构造函数 通过new关键字进行实例化
function Vue(options) {
  // 这里开始进行Vue初始化工作
  this._init(options);
}
// _init方法是挂载在Vue原型的方法 通过引入文件的方式进行原型挂载需要传入Vue
// 此做法有利于代码分割
initMixin(Vue);

// 混入_render
renderMixin(Vue);
// 混入_update
lifecycleMixin(Vue);
export default Vue;
复制代码
```

最后就是把定义在原型的方法引入到 Vue 主文件入口 这样所有的实例都能共享方法了

#### 6.模板编译的思维导图

![初始渲染](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a139db32c1d4f388707db685b177a42~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的初始渲染原理已经完结 结合前两篇响应式数据和模板编译 那么这时候我们已经可以把自己写好的模板渲染到页面了 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言

### 前言

此篇主要手写 Vue2.0 源码-**渲染更新原理**

上一篇咱们主要介绍了 Vue [初始渲染原理](https://juejin.cn/post/6937120983765483528) 完成了数据到视图层的映射过程 但是当我们改变数据的时候发现页面并不会自动更新 我们知道 Vue 的一个特性就是数据驱动 当数据改变的时候 我们无需手动操作 dom 视图会自动更新 回顾第一篇 [响应式数据原理](https://juejin.cn/post/6935344605424517128) 此篇主要采用观察者模式 定义 Watcher 和 Dep 完成依赖收集和派发更新 从而实现渲染更新

**适用人群：** 没时间去看官方源码或者看源码看的比较懵而不想去看的同学

> 提示：此篇难度稍大 是整个 Vue 源码非常核心的内容 后续的计算属性和自定义 watcher 以及$set $delete 等 Api 的实现 都需要理解此篇的思路 小编看源码这块也是看了有好几遍才搞懂 希望大家克服困难一起去实现一遍吧！

------

### 正文

```javascript
    <script>
      // Vue实例化
      let vm = new Vue({
        el: "#app",
        data() {
          return {
            a: 123,
          };
        },
        // render(h) {
        //   return h('div',{id:'a'},'hello')
        // },
        template: `<div id="a">hello {{a}}</div>`,
      });

        // 我们在这里模拟更新
      setTimeout(() => {
        vm.a = 456;
        // 此方法是刷新视图的核心
        vm._update(vm._render());
      }, 1000);
    </script>
复制代码
```

上段代码 我们在 setTimeout 里面调用 vm._update(vm._render())来实现更新功能 因为从上一篇初始渲染的原理可知 此方法就是渲染的核心 但是我们不可能每次数据变化都要求用户自己去调用渲染方法更新视图 我们需要一个机制在数据变动的时候自动去更新

#### 1.定义 Watcher

```javascript
// src/observer/watcher.js

// 全局变量id  每次new Watcher都会自增
let id = 0;

export default class Watcher {
  constructor(vm, exprOrFn, cb, options) {
    this.vm = vm;
    this.exprOrFn = exprOrFn;
    this.cb = cb; //回调函数 比如在watcher更新之前可以执行beforeUpdate方法
    this.options = options; //额外的选项 true代表渲染watcher
    this.id = id++; // watcher的唯一标识
    // 如果表达式是一个函数
    if (typeof exprOrFn === "function") {
      this.getter = exprOrFn;
    }
    // 实例化就会默认调用get方法
    this.get();
  }
  get() {
    this.getter();
  }
}
复制代码
```

在 observer 文件夹下新建 watcher.js 代表和观察者相关 这里首先介绍 Vue 里面使用到的[观察者模式](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fzhengyufeng%2Fp%2F10985321.html) 我们可以把 Watcher 当做观察者 它需要订阅数据的变动 当数据变动之后 通知它去执行某些方法 其实本质就是一个构造函数 初始化的时候会去执行 get 方法

#### 2.创建渲染 Watcher

```javascript
// src/lifecycle.js
export function mountComponent(vm, el) {
  //   _update和._render方法都是挂载在Vue原型的方法  类似_init

  // 引入watcher的概念 这里注册一个渲染watcher 执行vm._update(vm._render())方法渲染视图

  let updateComponent = () => {
    console.log("刷新页面");
    vm._update(vm._render());
  };
  new Watcher(vm, updateComponent, null, true);
}
复制代码
```

我们在组件挂载方法里面 定义一个渲染 Watcher 主要功能就是执行核心渲染页面的方法

#### 3.定义 Dep

```javascript
// src/observer/dep.js

// dep和watcher是多对多的关系

// 每个属性都有自己的dep

let id = 0; //dep实例的唯一标识
export default class Dep {
  constructor() {
    this.id = id++;
    this.subs = []; // 这个是存放watcher的容器
  }
}
// 默认Dep.target为null
Dep.target = null;
复制代码
```

Dep 也是一个构造函数 可以把他理解为观察者模式里面的被观察者 在 subs 里面收集 watcher 当数据变动的时候通知自身 subs 所有的 watcher 更新

Dep.target 是一个全局 Watcher 指向 初始状态是 null

#### 4.对象的依赖收集

```javascript
// src/observer/index.js

// Object.defineProperty数据劫持核心 兼容性在ie9以及以上
function defineReactive(data, key, value) {
  observe(value);

  let dep = new Dep(); // 为每个属性实例化一个Dep

  Object.defineProperty(data, key, {
    get() {
      // 页面取值的时候 可以把watcher收集到dep里面--依赖收集
      if (Dep.target) {
        // 如果有watcher dep就会保存watcher 同时watcher也会保存dep
        dep.depend();
      }
      return value;
    },
    set(newValue) {
      if (newValue === value) return;
      // 如果赋值的新值也是一个对象  需要观测
      observe(newValue);
      value = newValue;
      dep.notify(); // 通知渲染watcher去更新--派发更新
    },
  });
}
复制代码
```

上诉代码就是依赖收集和派发更新的核心 其实就是在数据被访问的时候 把我们定义好的渲染 Watcher 放到 dep 的 subs 数组里面 同时把 dep 实例对象也放到渲染 Watcher 里面去 数据更新时就可以通知 dep 的 subs 存储的 watcher 更新

#### 5.完善 watcher

```javascript
// src/observer/watcher.js

import { pushTarget, popTarget } from "./dep";

// 全局变量id  每次new Watcher都会自增
let id = 0;

export default class Watcher {
  constructor(vm, exprOrFn, cb, options) {
    this.vm = vm;
    this.exprOrFn = exprOrFn;
    this.cb = cb; //回调函数 比如在watcher更新之前可以执行beforeUpdate方法
    this.options = options; //额外的选项 true代表渲染watcher
    this.id = id++; // watcher的唯一标识
    this.deps = []; //存放dep的容器
    this.depsId = new Set(); //用来去重dep
    // 如果表达式是一个函数
    if (typeof exprOrFn === "function") {
      this.getter = exprOrFn;
    }
    // 实例化就会默认调用get方法
    this.get();
  }
  get() {
    pushTarget(this); // 在调用方法之前先把当前watcher实例推到全局Dep.target上
    this.getter(); //如果watcher是渲染watcher 那么就相当于执行  vm._update(vm._render()) 这个方法在render函数执行的时候会取值 从而实现依赖收集
    popTarget(); // 在调用方法之后把当前watcher实例从全局Dep.target移除
  }
  //   把dep放到deps里面 同时保证同一个dep只被保存到watcher一次  同样的  同一个watcher也只会保存在dep一次
  addDep(dep) {
    let id = dep.id;
    if (!this.depsId.has(id)) {
      this.depsId.add(id);
      this.deps.push(dep);
      //   直接调用dep的addSub方法  把自己--watcher实例添加到dep的subs容器里面
      dep.addSub(this);
    }
  }
  //   这里简单的就执行以下get方法  之后涉及到计算属性就不一样了
  update() {
    this.get();
  }
}
复制代码
```

watcher 在调用 getter 方法前后分别把自身赋值给 Dep.target 方便进行依赖收集 update 方法用来更新

#### 6.完善 dep

```javascript
// src/observer/dep.js

// dep和watcher是多对多的关系
// 每个属性都有自己的dep
let id = 0; //dep实例的唯一标识
export default class Dep {
  constructor() {
    this.id = id++;
    this.subs = []; // 这个是存放watcher的容器
  }
  depend() {
    //   如果当前存在watcher
    if (Dep.target) {
      Dep.target.addDep(this); // 把自身-dep实例存放在watcher里面
    }
  }
  notify() {
    //   依次执行subs里面的watcher更新方法
    this.subs.forEach((watcher) => watcher.update());
  }
  addSub(watcher) {
    //   把watcher加入到自身的subs容器
    this.subs.push(watcher);
  }
}
// 默认Dep.target为null
Dep.target = null;
// 栈结构用来存watcher
const targetStack = [];

export function pushTarget(watcher) {
  targetStack.push(watcher);
  Dep.target = watcher; // Dep.target指向当前watcher
}
export function popTarget() {
  targetStack.pop(); // 当前watcher出栈 拿到上一个watcher
  Dep.target = targetStack[targetStack.length - 1];
}
复制代码
```

定义相关的方法把收集依赖的同时把自身也放到 watcher 的 deps 容器里面去

> 思考？ 这时对象的更新已经可以满足了 但是如果是数组 类似{a:[1,2,3]} a.push(4) 并不会触发自动更新 因为我们数组并没有收集依赖

#### 7.数组的依赖收集

```javascript
// src/observer/index.js

// Object.defineProperty数据劫持核心 兼容性在ie9以及以上
function defineReactive(data, key, value) {
  let childOb = observe(value); // childOb就是Observer实例

  let dep = new Dep(); // 为每个属性实例化一个Dep

  Object.defineProperty(data, key, {
    get() {
      // 页面取值的时候 可以把watcher收集到dep里面--依赖收集
      if (Dep.target) {
        // 如果有watcher dep就会保存watcher 同时watcher也会保存dep
        dep.depend();
        if (childOb) {
          // 这里表示 属性的值依然是一个对象 包含数组和对象 childOb指代的就是Observer实例对象  里面的dep进行依赖收集
          // 比如{a:[1,2,3]} 属性a对应的值是一个数组 观测数组的返回值就是对应数组的Observer实例对象
          childOb.dep.depend();
          if (Array.isArray(value)) {
            // 如果数据结构类似 {a:[1,2,[3,4,[5,6]]]} 这种数组多层嵌套  数组包含数组的情况  那么我们访问a的时候 只是对第一层的数组进行了依赖收集 里面的数组因为没访问到  所以五大收集依赖  但是如果我们改变了a里面的第二层数组的值  是需要更新页面的  所以需要对数组递归进行依赖收集
            if (Array.isArray(value)) {
              // 如果内部还是数组
              dependArray(value); // 不停的进行依赖收集
            }
          }
        }
      }
      return value;
    },
    set(newValue) {
      if (newValue === value) return;
      // 如果赋值的新值也是一个对象  需要观测
      observe(newValue);
      value = newValue;
      dep.notify(); // 通知渲染watcher去更新--派发更新
    },
  });
}
// 递归收集数组依赖
function dependArray(value) {
  for (let e, i = 0, l = value.length; i < l; i++) {
    e = value[i];
    // e.__ob__代表e已经被响应式观测了 但是没有收集依赖 所以把他们收集到自己的Observer实例的dep里面
    e && e.__ob__ && e.__ob__.dep.depend();
    if (Array.isArray(e)) {
      // 如果数组里面还有数组  就递归去收集依赖
      dependArray(e);
    }
  }
}
复制代码
```

如果对象属性的值是一个数组 那么执行 childOb.dep.depend()收集数组的依赖 如果数组里面还包含数组 需要递归遍历收集 因为只有访问数据触发了 get 才会去收集依赖 一开始只是递归对数据进行响应式处理无法收集依赖 这两点需要分清

#### 8.数组的派发更新

```javascript
// src/observer/array.js

methodsToPatch.forEach((method) => {
  arrayMethods[method] = function (...args) {
    //   这里保留原型方法的执行结果
    const result = arrayProto[method].apply(this, args);
    // 这句话是关键
    // this代表的就是数据本身 比如数据是{a:[1,2,3]} 那么我们使用a.push(4)  this就是a  ob就是a.__ob__ 这个属性代表的是该数据已经被响应式观察过了 __ob__对象指的就是Observer实例
    const ob = this.__ob__;
    let inserted;
    switch (method) {
      case "push":
      case "unshift":
        inserted = args;
        break;
      case "splice":
        inserted = args.slice(2);
      default:
        break;
    }
    if (inserted) ob.observeArray(inserted); // 对新增的每一项进行观测
    ob.dep.notify(); //数组派发更新 ob指的就是数组对应的Observer实例 我们在get的时候判断如果属性的值还是对象那么就在Observer实例的dep收集依赖 所以这里是一一对应的  可以直接更新
    return result;
  };
});
复制代码
```

关键代码就是 ob.dep.notify()

#### 9.渲染更新的思维导图

![渲染更新](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/677a42dfc3b64ccf9fe4ad536ede74f3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

![整体流程](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2351d651b8384153b19215a0297fd6e6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

这里放一张 整个 Vue 响应式原理的图片 咱们从数据劫持-->模板解析-->模板渲染-->数据变化视图自动更新整个流程已经手写了一遍 尤其是此篇介绍的渲染更新相关的知识点 建议反复理解原理之后自己动手实现一遍 因为Vue 很多核心原理和 api 都跟这里的知识点相关哈

至此 Vue 的渲染更新原理已经完结 遇到不懂或者有争议的地方欢迎评论留言

此篇主要手写 Vue2.0 源码-**异步更新原理**

上一篇咱们主要介绍了 Vue [渲染更新原理](https://juejin.cn/post/6938221715281575973) 咱们已经可以实现数据改变 视图自动更新了 那么此篇主要是对视图更新的性能优化 包含 nextTick 这一重要的 api 实现

**适用人群：** 没时间去看官方源码或者看源码看的比较懵而不想去看的同学

> 建议：本文涉及到 js 事件循环相关的概念 对宏任务和微任务不是很清楚的同学可以先看看相关资料

------

### 正文

```html
<script>
  // Vue实例化
  let vm = new Vue({
    el: "#app",
    data() {
      return {
        a: 123,
      };
    },
    // render(h) {
    //   return h('div',{id:'a'},'hello')
    // },
    template: `<div id="a">hello {{a}}</div>`,
  });

  // 当我们每一次改变数据的时候  渲染watcher都会执行一次 这个是影响性能的
  setTimeout(() => {
    vm.a = 1;
    vm.a = 2;
    vm.a = 3;
  }, 1000);
</script>
复制代码
```

大家思考一下 按照之前的逻辑 每次我们改变数据的时候都会触发相应的 watcher 进行更新 如果是渲染 watcher 那是不是意味着 数据变动一次 就会重新渲染一次 这样其实是很浪费性能的 我们有没有更好的方法 让数据变动完毕后统一去更新视图呢

#### 1.watcher 更新的改写

```javascript
// src/observer/watcher.js

import { queueWatcher } from "./scheduler";
export default class Watcher {
  update() {
    // 每次watcher进行更新的时候  是否可以让他们先缓存起来  之后再一起调用
    // 异步队列机制
    queueWatcher(this);
  }
  run() {
    // 真正的触发更新
    this.get();
  }
}
复制代码
```

我们把 update 更新方法改一下 增加异步队列的机制

#### 2.queueWatcher 实现队列机制

```javascript
// src/observer/scheduler.js

import { nextTick } from "../util/next-tick";
let queue = [];
let has = {};
function flushSchedulerQueue() {
  for (let index = 0; index < queue.length; index++) {
    //   调用watcher的run方法 执行真正的更新操作
    queue[index].run();
  }
  // 执行完之后清空队列
  queue = [];
  has = {};
}

// 实现异步队列机制
export function queueWatcher(watcher) {
  const id = watcher.id;
  //   watcher去重
  if (has[id] === undefined) {
    //  同步代码执行 把全部的watcher都放到队列里面去
    queue.push(watcher);
    has[id] = true;
    // 进行异步调用
    nextTick(flushSchedulerQueue);
  }
}
}
复制代码
```

新建 scheduler.js 文件 表示和调度相关 先同步把 watcher 都放到队列里面去 执行完队列的事件之后再清空队列 主要使用 nextTick 来执行 watcher 队列

#### 3.nextTick 实现原理

```javascript
// src/util/next-tick.js

let callbacks = [];
let pending = false;
function flushCallbacks() {
  pending = false; //把标志还原为false
  // 依次执行回调
  for (let i = 0; i < callbacks.length; i++) {
    callbacks[i]();
  }
}
let timerFunc; //定义异步方法  采用优雅降级
if (typeof Promise !== "undefined") {
  // 如果支持promise
  const p = Promise.resolve();
  timerFunc = () => {
    p.then(flushCallbacks);
  };
} else if (typeof MutationObserver !== "undefined") {
  // MutationObserver 主要是监听dom变化 也是一个异步方法
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true,
  });
  timerFunc = () => {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
} else if (typeof setImmediate !== "undefined") {
  // 如果前面都不支持 判断setImmediate
  timerFunc = () => {
    setImmediate(flushCallbacks);
  };
} else {
  // 最后降级采用setTimeout
  timerFunc = () => {
    setTimeout(flushCallbacks, 0);
  };
}

export function nextTick(cb) {
  // 除了渲染watcher  还有用户自己手动调用的nextTick 一起被收集到数组
  callbacks.push(cb);
  if (!pending) {
    // 如果多次调用nextTick  只会执行一次异步 等异步队列清空之后再把标志变为false
    pending = true;
    timerFunc();
  }
}
复制代码
```

新建 util/next-tick.js 代表工具类函数 因为 nextTick 用户也可以手动调用 主要思路就是采用微任务优先的方式调用异步方法去执行 nextTick 包装的方法

#### 4.$nextTick 挂载原型

```javascript
// src/render.js

import { nextTick } from "./util/next-tick";

export function renderMixin(Vue) {
  // 挂载在原型的nextTick方法 可供用户手动调用
  Vue.prototype.$nextTick = nextTick;
}
复制代码
```

最后把$nextTick 挂载到 Vue 的原型

#### 4.异步更新的思维导图

![Vue2.0源码-异步更新.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7dfc33a82cbe41a3950da87812f7f54f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的异步更新原理已经完结 核心原理就是 nextTick 实现异步队列 前提是需要理解 js 事件循环机制 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言



此篇主要手写 Vue2.0 源码-**diff 算法原理**

上一篇咱们主要介绍了 Vue [异步更新原理](https://juejin.cn/post/6939704519668432910) 是对视图更新的性能优化 此篇同样是对渲染更新的优化 当模板发生变化之后 我们可以利用 diff 算法 对比新老虚拟 dom 看是否能进行节点复用 diff 算法也是 vue 面试比较热门的考点哈

**适用人群：**

1.想要深入理解 vue 源码更好的进行日常业务开发

2.想要在简历写上精通 vue 框架源码（再也不怕面试官的连环夺命问 哈哈）

3.没时间去看官方源码或者初看源码觉得难以理解的同学

------

### 正文

```html
<script>
  // Vue实例化
  let vm = new Vue({
    el: "#app",
    data() {
      return {
        a: 123,
      };
    },
    template: `<div id="a">hello {{a}}</div>`,
  });

  setTimeout(() => {
    vm.a = 1;
  }, 1000);
</script>
复制代码
```

大家思考一下 如果我们当初始渲染完成 1 秒后改变了一下模板里面 a 的值 vue 会怎么处理来显示最新的值呢?

1.把上次渲染的真实 dom 删除 然后重新渲染一个新的 dom 节点来应用最新的 a 的值

2.把老的 dom 进行复用 改变一下内部文本节点的 textContent 的值

> 这两种方案 很明显后者的性能开销更小 一起来看看 vue 怎么使用 diff 算法来进行渲染更新的吧

#### 1.patch 核心渲染方法改写

```javascript
// src/vdom/patch.js

export function patch(oldVnode, vnode) {
  const isRealElement = oldVnode.nodeType;
  if (isRealElement) {
    // oldVnode是真实dom元素 就代表初次渲染
  } else {
    // oldVnode是虚拟dom 就是更新过程 使用diff算法
    if (oldVnode.tag !== vnode.tag) {
      // 如果新旧标签不一致 用新的替换旧的 oldVnode.el代表的是真实dom节点--同级比较
      oldVnode.el.parentNode.replaceChild(createElm(vnode), oldVnode.el);
    }
    // 如果旧节点是一个文本节点
    if (!oldVnode.tag) {
      if (oldVnode.text !== vnode.text) {
        oldVnode.el.textContent = vnode.text;
      }
    }
    // 不符合上面两种 代表标签一致 并且不是文本节点
    // 为了节点复用 所以直接把旧的虚拟dom对应的真实dom赋值给新的虚拟dom的el属性
    const el = (vnode.el = oldVnode.el);
    updateProperties(vnode, oldVnode.data); // 更新属性
    const oldCh = oldVnode.children || []; // 老的儿子
    const newCh = vnode.children || []; // 新的儿子
    if (oldCh.length > 0 && newCh.length > 0) {
      // 新老都存在子节点
      updateChildren(el, oldCh, newCh);
    } else if (oldCh.length) {
      // 老的有儿子新的没有
      el.innerHTML = "";
    } else if (newCh.length) {
      // 新的有儿子
      for (let i = 0; i < newCh.length; i++) {
        const child = newCh[i];
        el.appendChild(createElm(child));
      }
    }
  }
}
复制代码
```

我们直接看 else 分支 代表的是渲染更新过程 可以分为以下几步

1.diff 只进行同级比较 ![diff同级对比.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a856bf3d3fa49698f70122bc0c09ab3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

2.根据新老 vnode 子节点不同情况分别处理 ![diff流程.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90304a24ceab464abe2eafcef9490091~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 2.updateProperties 更新属性

```javascript
//  src/vdom/patch.js

// 解析vnode的data属性 映射到真实dom上
function updateProperties(vnode, oldProps = {}) {
  const newProps = vnode.data || {}; //新的vnode的属性
  const el = vnode.el; // 真实节点
  // 如果新的节点没有 需要把老的节点属性移除
  for (const k in oldProps) {
    if (!newProps[k]) {
      el.removeAttribute(k);
    }
  }
  // 对style样式做特殊处理 如果新的没有 需要把老的style值置为空
  const newStyle = newProps.style || {};
  const oldStyle = oldProps.style || {};
  for (const key in oldStyle) {
    if (!newStyle[key]) {
      el.style[key] = "";
    }
  }
  // 遍历新的属性 进行增加操作
  for (const key in newProps) {
    if (key === "style") {
      for (const styleName in newProps.style) {
        el.style[styleName] = newProps.style[styleName];
      }
    } else if (key === "class") {
      el.className = newProps.class;
    } else {
      // 给这个元素添加属性 值就是对应的值
      el.setAttribute(key, newProps[key]);
    }
  }
}
复制代码
```

对比新老 vnode 进行属性更新

#### 3.updateChildren 更新子节点-diff 核心方法

```javascript
// src/vdom/patch.js

// 判断两个vnode的标签和key是否相同 如果相同 就可以认为是同一节点就地复用
function isSameVnode(oldVnode, newVnode) {
  return oldVnode.tag === newVnode.tag && oldVnode.key === newVnode.key;
}
// diff算法核心 采用双指针的方式 对比新老vnode的儿子节点
function updateChildren(parent, oldCh, newCh) {
  let oldStartIndex = 0; //老儿子的起始下标
  let oldStartVnode = oldCh[0]; //老儿子的第一个节点
  let oldEndIndex = oldCh.length - 1; //老儿子的结束下标
  let oldEndVnode = oldCh[oldEndIndex]; //老儿子的起结束节点

  let newStartIndex = 0; //同上  新儿子的
  let newStartVnode = newCh[0];
  let newEndIndex = newCh.length - 1;
  let newEndVnode = newCh[newEndIndex];

  // 根据key来创建老的儿子的index映射表  类似 {'a':0,'b':1} 代表key为'a'的节点在第一个位置 key为'b'的节点在第二个位置
  function makeIndexByKey(children) {
    let map = {};
    children.forEach((item, index) => {
      map[item.key] = index;
    });
    return map;
  }
  // 生成的映射表
  let map = makeIndexByKey(oldCh);

  // 只有当新老儿子的双指标的起始位置不大于结束位置的时候  才能循环 一方停止了就需要结束循环
  while (oldStartIndex <= oldEndIndex && newStartIndex <= newEndIndex) {
    // 因为暴力对比过程把移动的vnode置为 undefined 如果不存在vnode节点 直接跳过
    if (!oldStartVnode) {
      oldStartVnode = oldCh[++oldStartIndex];
    } else if (!oldEndVnode) {
      oldEndVnode = oldCh[--oldEndIndex];
    } else if (isSameVnode(oldStartVnode, newStartVnode)) {
      // 头和头对比 依次向后追加
      patch(oldStartVnode, newStartVnode); //递归比较儿子以及他们的子节点
      oldStartVnode = oldCh[++oldStartIndex];
      newStartVnode = newCh[++newStartIndex];
    } else if (isSameVnode(oldEndVnode, newEndVnode)) {
      //尾和尾对比 依次向前追加
      patch(oldEndVnode, newEndVnode);
      oldEndVnode = oldCh[--oldEndIndex];
      newEndVnode = newCh[--newEndIndex];
    } else if (isSameVnode(oldStartVnode, newEndVnode)) {
      // 老的头和新的尾相同 把老的头部移动到尾部
      patch(oldStartVnode, newEndVnode);
      parent.insertBefore(oldStartVnode.el, oldEndVnode.el.nextSibling); //insertBefore可以移动或者插入真实dom
      oldStartVnode = oldCh[++oldStartIndex];
      newEndVnode = newCh[--newEndIndex];
    } else if (isSameVnode(oldEndVnode, newStartVnode)) {
      // 老的尾和新的头相同 把老的尾部移动到头部
      patch(oldEndVnode, newStartVnode);
      parent.insertBefore(oldEndVnode.el, oldStartVnode.el);
      oldEndVnode = oldCh[--oldEndIndex];
      newStartVnode = newCh[++newStartIndex];
    } else {
      // 上述四种情况都不满足 那么需要暴力对比
      // 根据老的子节点的key和index的映射表 从新的开始子节点进行查找 如果可以找到就进行移动操作 如果找不到则直接进行插入
      let moveIndex = map[newStartVnode.key];
      if (!moveIndex) {
        // 老的节点找不到  直接插入
        parent.insertBefore(createElm(newStartVnode), oldStartVnode.el);
      } else {
        let moveVnode = oldCh[moveIndex]; //找得到就拿到老的节点
        oldCh[moveIndex] = undefined; //这个是占位操作 避免数组塌陷  防止老节点移动走了之后破坏了初始的映射表位置
        parent.insertBefore(moveVnode.el, oldStartVnode.el); //把找到的节点移动到最前面
        patch(moveVnode, newStartVnode);
      }
    }
  }
  // 如果老节点循环完毕了 但是新节点还有  证明  新节点需要被添加到头部或者尾部
  if (newStartIndex <= newEndIndex) {
    for (let i = newStartIndex; i <= newEndIndex; i++) {
      // 这是一个优化写法 insertBefore的第一个参数是null等同于appendChild作用
      const ele =
        newCh[newEndIndex + 1] == null ? null : newCh[newEndIndex + 1].el;
      parent.insertBefore(createElm(newCh[i]), ele);
    }
  }
  // 如果新节点循环完毕 老节点还有  证明老的节点需要直接被删除
  if (oldStartIndex <= oldEndIndex) {
    for (let i = oldStartIndex; i <= oldEndIndex; i++) {
      let child = oldCh[i];
      if (child != undefined) {
        parent.removeChild(child.el);
      }
    }
  }
}
复制代码
```

这段代码特别长 但是理解起来其实可以简单的分为以下几点

1.使用双指针移动来进行新老节点的对比 ![diff双指针.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d3aa437b5724cbfbdb0cf4503e7aa12~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

2.用 isSameVnode 来判断新老子节点的头头 尾尾 头尾 尾头 是否是同一节点 如果满足就进行相应的移动指针(头头 尾尾)或者移动 dom 节点(头尾 尾头)操作

3.如果全都不相等 进行暴力对比 如果找到了利用 key 和 index 的映射表来移动老的子节点到前面去 如果找不到就直接插入

![diff暴力对比.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c790b59c7c6f4708912be3598c42811f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

4.对老的子节点进行递归 patch 处理

5.最后老的子节点有多的就删掉 新的子节点有多的就添加到相应的位置

#### 4.改造原型渲染更新方法_update

```javascript
// src/lifecycle.js

export function lifecycleMixin(Vue) {
  // 把_update挂载在Vue的原型
  Vue.prototype._update = function (vnode) {
    const vm = this;
    const prevVnode = vm._vnode; // 保留上一次的vnode
    vm._vnode = vnode;
    if (!prevVnode) {
      // patch是渲染vnode为真实dom核心
      vm.$el = patch(vm.$el, vnode); // 初次渲染 vm._vnode肯定不存在 要通过虚拟节点 渲染出真实的dom 赋值给$el属性
    } else {
      vm.$el = patch(prevVnode, vnode); // 更新时把上次的vnode和这次更新的vnode穿进去 进行diff算法
    }
  };
}
复制代码
```

改造_update 方法 在 Vue 实例的_vnode 保留上次的 vnode 节点 以供 patch 进行新老虚拟 dom 的对比

#### 5.diff 算法的思维导图

![Vue2.0源码-diff算法.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f964567d4dbc4228bf9bdbdb4ad70c06~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的 diff 算法源码已经完结 大家主要理解 diff 整个过程 此篇代码量和难度稍微偏大 建议大家反复看 因为 diff 算法是 vue 里面非常核心的知识点 也是面试的常考点 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言

此篇主要手写 Vue2.0 源码-**Mixin 混入原理**

上一篇咱们主要介绍了 Vue [diff算法原理 ](https://juejin.cn/post/6953433215218483236) 核心是运用 diff算法来进行渲染优化 此篇主要包含 Mixin 混入 这是 Vue 里面非常关键的一个 api 在 Vue 初始化的时候起到了合并选项的重要作用

**适用人群：**

1.想要深入理解 vue 源码更好的进行日常业务开发

2.想要在简历写上精通 vue 框架源码（再也不怕面试官的连环夺命问 哈哈）

3.没时间去看官方源码或者初看源码觉得难以理解的同学

------

### 正文

```javascript
Vue.mixin({
  created() {
    console.log("我是全局混入");
  },
});

// Vue实例化
let vm = new Vue({
  el: "#app",
  data() {
    return {
      a: { a: { a: { b: 456 } } },
      aa: 1,
      bb: 2,
    };
  },
  created() {
    console.log("我是自己的");
  },
  template: `<div id="a">hello 这是我自己写的Vue{{name}}
          </div>`,
});
复制代码
```

当我们在 Vue 里面想要复用一段业务代码逻辑时经常用到的就是混入的方法 但是对于混入的原理 混入的先后顺序以及不同选项的合并策略大家是否都清楚呢 让我们一起来手写一遍就都清楚了

#### 1.定义全局 Mixin 函数

```javascript
// src/global-api/mixin.js

import {mergeOptions} from '../util/index'
export default function initMixin(Vue){
  Vue.mixin = function (mixin) {
    //   合并对象
      this.options=mergeOptions(this.options,mixin)
  };
}
};
复制代码
```

新建 global-api 文件夹 把 mixin 定义为 Vue 的全局方法 核心方法就是利用 mergeOptions 把传入的选项混入到自己的 options 上面

```javascript
// src/index.js
import { initMixin } from "./init.js";
// Vue就是一个构造函数 通过new关键字进行实例化
function Vue(options) {
  // 这里开始进行Vue初始化工作
  this._init(options);
}
// 此做法有利于代码分割
initMixin(Vue);
export default Vue;
复制代码
```

然后在 Vue 的入口文件里面引入 initMixin 方法

#### 2.mergeOptions 方法

```javascript
// src/util/index.js
// 定义生命周期
export const LIFECYCLE_HOOKS = [
  "beforeCreate",
  "created",
  "beforeMount",
  "mounted",
  "beforeUpdate",
  "updated",
  "beforeDestroy",
  "destroyed",
];

// 合并策略
const strats = {};

//生命周期合并策略
function mergeHook(parentVal, childVal) {
  // 如果有儿子
  if (childVal) {
    if (parentVal) {
      // 合并成一个数组
      return parentVal.concat(childVal);
    } else {
      // 包装成一个数组
      return [childVal];
    }
  } else {
    return parentVal;
  }
}

// 为生命周期添加合并策略
LIFECYCLE_HOOKS.forEach((hook) => {
  strats[hook] = mergeHook;
});

// mixin核心方法
export function mergeOptions(parent, child) {
  const options = {};
  // 遍历父亲
  for (let k in parent) {
    mergeFiled(k);
  }
  // 父亲没有 儿子有
  for (let k in child) {
    if (!parent.hasOwnProperty(k)) {
      mergeFiled(k);
    }
  }

  //真正合并字段方法
  function mergeFiled(k) {
    if (strats[k]) {
      options[k] = strats[k](parent[k], child[k]);
    } else {
      // 默认策略
      options[k] = child[k] ? child[k] : parent[k];
    }
  }
  return options;
}
复制代码
```

我们先着重看下 mergeOptions 方法 主要是遍历父亲和儿子的属性 进行合并 如果合并的选项有自己的合并策略 那么就是用相应的合并策略

再来看看 我们这里的生命周期的合并策略 mergeHook 很明显是把全部的生命周期都各自混入成了数组的形式依次调用

#### 3.生命周期的调用

```javascript
// src/lifecycle.js

export function callHook(vm, hook) {
  // 依次执行生命周期对应的方法
  const handlers = vm.$options[hook];
  if (handlers) {
    for (let i = 0; i < handlers.length; i++) {
      handlers[i].call(vm); //生命周期里面的this指向当前实例
    }
  }
}
复制代码
```

把$options 上面的生命周期依次遍历进行调用

```javascript
// src/init.js

Vue.prototype._init = function (options) {
  const vm = this;
  // 这里的this代表调用_init方法的对象(实例对象)
  //  this.$options就是用户new Vue的时候传入的属性和全局的Vue.options合并之后的结果

  vm.$options = mergeOptions(vm.constructor.options, options);
  callHook(vm, "beforeCreate"); //初始化数据之前
  // 初始化状态
  initState(vm);
  callHook(vm, "created"); //初始化数据之后
  // 如果有el属性 进行模板渲染
  if (vm.$options.el) {
    vm.$mount(vm.$options.el);
  }
};
复制代码
```

在 init 初始化的时候调用 mergeOptions 来进行选项合并 之后在需要调用生命周期的地方运用 callHook 来执行用户传入的相关方法

```javascript
// src/lifecycle.js
export function mountComponent(vm, el) {
  vm.$el = el;
  // 引入watcher的概念 这里注册一个渲染watcher 执行vm._update(vm._render())方法渲染视图
  callHook(vm, "beforeMount"); //初始渲染之前
  let updateComponent = () => {
    vm._update(vm._render());
  };
  new Watcher(
    vm,
    updateComponent,
    () => {
      callHook(vm, "beforeUpdate"); //更新之前
    },
    true
  );
  callHook(vm, "mounted"); //渲染完成之后
}
复制代码
```

在 mountComponent 方法里面调用相关的生命周期 callHook

#### 4.混入的思维导图

![Vue2.0源码-mixin原理.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80ad9361be64258b70c578cb3b14b74~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的混入原型已经手写完毕 其实最核心的就是对象合并以及不同选项的合并策略 目前只是演示了生命周期的合并策略 后续到组件的时候会讲到组件相关的合并策略 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言

此篇主要手写 Vue2.0 源码-**组件原理**

上一篇咱们主要介绍了 Vue [Mixin原理](https://juejin.cn/post/6951671158198501383) 是Vue初始化选项合并核心的api 大家都知道 Vue 的一大特色就是组件化 此篇主要介绍整个组件创建和渲染流程 其中 Vue.extend 这一 api 是创建组件的核心

**适用人群：**

1.想要深入理解 vue 源码更好的进行日常业务开发

2.想要在简历写上精通 vue 框架源码（再也不怕面试官的连环夺命问 哈哈）

3.没时间去看官方源码或者初看源码觉得难以理解的同学

------

### 正文

```html
<script>
  // 全局组件
  Vue.component("parent-component", {
    template: `<div>我是全局组件</div>`,
  });
  // Vue实例化
  let vm = new Vue({
    el: "#app",
    data() {
      return {
        aa: 1,
      };
    },
    // render(h) {
    //   return h('div',{id:'a'},'hello')
    // },
    template: `<div id="a">
      hello 这是我自己写的Vue{{aa}}
      <parent-component><parent-component>
      <child-component></child-component>
      </div>`,
    // 局部组件
    components: {
      "child-component": {
        template: `<div>我是局部组件</div>`,
      },
    },
  });
</script>
复制代码
```

上面演示了最基础的全局组件和局部组件的用法 其实我们每一个组件都是一个继承自 Vue 的子类 能够使用 Vue 的原型方法

#### 1.全局组件注册

```javascript
// src/global-api/index.js

import initExtend from "./initExtend";
import initAssetRegisters from "./assets";
const ASSETS_TYPE = ["component", "directive", "filter"];
export function initGlobalApi(Vue) {
  Vue.options = {}; // 全局的组件 指令 过滤器
  ASSETS_TYPE.forEach((type) => {
    Vue.options[type + "s"] = {};
  });
  Vue.options._base = Vue; //_base指向Vue

  initExtend(Vue); // extend方法定义
  initAssetRegisters(Vue); //assets注册方法 包含组件 指令和过滤器
}
复制代码
```

initGlobalApi方法主要用来注册Vue的全局方法 比如之前写的Vue.Mixin 和今天的Vue.extend Vue.component等

```javascript
// src/global-api/asset.js

const ASSETS_TYPE = ["component", "directive", "filter"];
export default function initAssetRegisters(Vue) {
  ASSETS_TYPE.forEach((type) => {
    Vue[type] = function (id, definition) {
      if (type === "component") {
        //   this指向Vue
        // 全局组件注册
        // 子组件可能也有extend方法  VueComponent.component方法
        definition = this.options._base.extend(definition);
      }
      this.options[type + "s"][id] = definition;
    };
  });
}
复制代码
```

this.options._base 就是指代 Vue 可见所谓的全局组件就是使用 Vue.extend 方法把传入的选项处理之后挂载到了 Vue.options.components 上面

#### 2.Vue.extend 定义

```javascript
//  src/global-api/initExtend.js

import { mergeOptions } from "../util/index";
export default function initExtend(Vue) {
  let cid = 0; //组件的唯一标识
  // 创建子类继承Vue父类 便于属性扩展
  Vue.extend = function (extendOptions) {
    // 创建子类的构造函数 并且调用初始化方法
    const Sub = function VueComponent(options) {
      this._init(options); //调用Vue初始化方法
    };
    Sub.cid = cid++;
    Sub.prototype = Object.create(this.prototype); // 子类原型指向父类
    Sub.prototype.constructor = Sub; //constructor指向自己
    Sub.options = mergeOptions(this.options, extendOptions); //合并自己的options和父类的options
    return Sub;
  };
}
复制代码
```

Vue.extend 核心思路就是使用原型继承的方法返回了 Vue 的子类 并且利用 mergeOptions 把传入组件的 options 和父类的 options 进行了合并

#### 3.组件的合并策略

```javascript
// src/init.js

Vue.prototype._init = function (options) {
  const vm = this;
  vm.$options = mergeOptions(vm.constructor.options, options); //合并options
};
复制代码
```

还记得我们 Vue 初始化的时候合并 options 吗 全局组件挂载在 Vue.options.components 上 局部组件也定义在自己的 options.components 上面 那我们怎么处理全局组件和局部组件的合并呢

```javascript
// src/util/index.js

const ASSETS_TYPE = ["component", "directive", "filter"];
// 组件 指令 过滤器的合并策略
function mergeAssets(parentVal, childVal) {
  const res = Object.create(parentVal); //比如有同名的全局组件和自己定义的局部组件 那么parentVal代表全局组件 自己定义的组件是childVal  首先会查找自已局部组件有就用自己的  没有就从原型继承全局组件  res.__proto__===parentVal
  if (childVal) {
    for (let k in childVal) {
      res[k] = childVal[k];
    }
  }
  return res;
}

// 定义组件的合并策略
ASSETS_TYPE.forEach((type) => {
  strats[type + "s"] = mergeAssets;
});
复制代码
```

这里又使用到了原型继承的方式来进行组件合并 组件内部优先查找自己局部定义的组件 找不到会向上查找原型中定义的组件

#### 4.创建组件 Vnode

```javascript
// src/util/index.js

export function isObject(data) {
  //判断是否是对象
  if (typeof data !== "object" || data == null) {
    return false;
  }
  return true;
}

export function isReservedTag(tagName) {
  //判断是不是常规html标签
  // 定义常见标签
  let str =
    "html,body,base,head,link,meta,style,title," +
    "address,article,aside,footer,header,h1,h2,h3,h4,h5,h6,hgroup,nav,section," +
    "div,dd,dl,dt,figcaption,figure,picture,hr,img,li,main,ol,p,pre,ul," +
    "a,b,abbr,bdi,bdo,br,cite,code,data,dfn,em,i,kbd,mark,q,rp,rt,rtc,ruby," +
    "s,samp,small,span,strong,sub,sup,time,u,var,wbr,area,audio,map,track,video," +
    "embed,object,param,source,canvas,script,noscript,del,ins," +
    "caption,col,colgroup,table,thead,tbody,td,th,tr," +
    "button,datalist,fieldset,form,input,label,legend,meter,optgroup,option," +
    "output,progress,select,textarea," +
    "details,dialog,menu,menuitem,summary," +
    "content,element,shadow,template,blockquote,iframe,tfoot";
  let obj = {};
  str.split(",").forEach((tag) => {
    obj[tag] = true;
  });
  return obj[tagName];
}
复制代码
```

上诉是公用工具方法 在创建组件 Vnode 过程会用到

```javascript
// src/vdom/index.js

import { isObject, isReservedTag } from "../util/index";
// 创建元素vnode 等于render函数里面的 h=>h(App)
export function createElement(vm, tag, data = {}, ...children) {
  let key = data.key;

  if (isReservedTag(tag)) {
    // 如果是普通标签
    return new Vnode(tag, data, key, children);
  } else {
    // 否则就是组件
    let Ctor = vm.$options.components[tag]; //获取组件的构造函数
    return createComponent(vm, tag, data, key, children, Ctor);
  }
}

function createComponent(vm, tag, data, key, children, Ctor) {
  if (isObject(Ctor)) {
    //   如果没有被改造成构造函数
    Ctor = vm.$options._base.extend(Ctor);
  }
  // 声明组件自己内部的生命周期
  data.hook = {
    // 组件创建过程的自身初始化方法
    init(vnode) {
      let child = (vnode.componentInstance = new Ctor({ _isComponent: true })); //实例化组件
      child.$mount(); //因为没有传入el属性  需要手动挂载 为了在组件实例上面增加$el方法可用于生成组件的真实渲染节点
    },
  };

  // 组件vnode  也叫占位符vnode  ==> $vnode
  return new Vnode(
    `vue-component-${Ctor.cid}-${tag}`,
    data,
    key,
    undefined,
    undefined,
    {
      Ctor,
      children,
    }
  );
}
复制代码
```

改写 createElement 方法 对于非普通 html 标签 就需要生成组件 Vnode 把 Ctor 和 children 作为 Vnode 最后一个参数 componentOptions 传入

这里需要注意组件的 data.hook.init 方法 我们手动调用 child.$mount()方法 此方法可以生成组件的真实 dom 并且挂载到自身的 $el 属性上面 对此处有疑问的可以查看小编之前文章 [手写 Vue2.0 源码（三）-初始渲染原理](https://juejin.cn/post/6937120983765483528)

#### 5.渲染组件真实节点

```javascript
// src/vdom/patch.js

// patch用来渲染和更新视图
export function patch(oldVnode, vnode) {
  if (!oldVnode) {
    // 组件的创建过程是没有el属性的
    return createElm(vnode);
  } else {
    //   非组件创建过程省略
  }
}

// 判断是否是组件Vnode
function createComponent(vnode) {
  // 初始化组件
  // 创建组件实例
  let i = vnode.data;
  //   下面这句话很关键 调用组件data.hook.init方法进行组件初始化过程 最终组件的vnode.componentInstance.$el就是组件渲染好的真实dom
  if ((i = i.hook) && (i = i.init)) {
    i(vnode);
  }
  // 如果组件实例化完毕有componentInstance属性 那证明是组件
  if (vnode.componentInstance) {
    return true;
  }
}

// 虚拟dom转成真实dom
function createElm(vnode) {
  const { tag, data, key, children, text } = vnode;
  //   判断虚拟dom 是元素节点还是文本节点
  if (typeof tag === "string") {
    if (createComponent(vnode)) {
      // 如果是组件 返回真实组件渲染的真实dom
      return vnode.componentInstance.$el;
    }
    //   虚拟dom的el属性指向真实dom 方便后续更新diff算法操作
    vnode.el = document.createElement(tag);
    // 解析虚拟dom属性
    updateProperties(vnode);
    // 如果有子节点就递归插入到父节点里面
    children.forEach((child) => {
      return vnode.el.appendChild(createElm(child));
    });
  } else {
    //   文本节点
    vnode.el = document.createTextNode(text);
  }
  return vnode.el;
}
复制代码
```

判断如果属于组件 Vnode 那么把渲染好的组件真实 dom ==>vnode.componentInstance.$el 返回

#### 6.组件的思维导图

![Vue2.0源码-组件.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/396c604a21a3466c86f43ceb3ccf1b1f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的 组件源码已经完结 其实每一个组件都是一个个 Vue 的实例 都会经历 init 初始化方法 建议学习组件之前先把前面的系列搞懂 组件就比较容易理解了 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言



此篇主要手写 Vue2.0 源码-**侦听属性**

上一篇咱们主要介绍了 Vue [组件原理](https://juejin.cn/post/6954173708344770591) 深入了解了 Vue 组件化开发的特色 此篇将介绍我们日常业务开发使用非常多的侦听属性的原理

**适用人群：**

1.想要深入理解 vue 源码更好的进行日常业务开发

2.想要在简历写上精通 vue 框架源码（再也不怕面试官的连环夺命问 哈哈）

3.没时间去看官方源码或者初看源码觉得难以理解的同学

------

### 正文

```html
<script>
  // Vue实例化
  let vm = new Vue({
    el: "#app",
    data() {
      return {
        aa: 1,
        bb: 2,
      };
    },
    template: `<div id="a">hello 这是我自己写的Vue{{name}}</div>`,
    methods: {
      doSomething() {},
    },
    watch: {
      aa(newVal, oldVal) {
        console.log(newVal);
      },
      // aa: {
      //   handle(newVal， oldVal) {
      //     console.log(newVal);
      //   },
      //   deep: true
      // },
      // aa: 'doSomething',
      // aa: [{
      //   handle(newVal， oldVal) {
      //     console.log(newVal);
      //   },
      //   deep: true
      // }]
    },
  });
  setTimeout(() => {
    vm.aa = 1111;
  }, 1000);
</script>
复制代码
```

侦听属性的写法很多 可以写成 字符串 函数 数组 以及对象 对于对象的写法自己可以增加一些 options 用来增强功能 侦听属性的特点是监听的值发生了变化之后可以执行用户传入的自定义方法

#### 1.侦听属性的初始化

```javascript
// src/state.js

// 统一初始化数据的方法
export function initState(vm) {
  // 获取传入的数据对象
  const opts = vm.$options;
  if (opts.watch) {
    //侦听属性初始化
    initWatch(vm);
  }
}

// 初始化watch
function initWatch(vm) {
  let watch = vm.$options.watch;
  for (let k in watch) {
    const handler = watch[k]; //用户自定义watch的写法可能是数组 对象 函数 字符串
    if (Array.isArray(handler)) {
      // 如果是数组就遍历进行创建
      handler.forEach((handle) => {
        createWatcher(vm, k, handle);
      });
    } else {
      createWatcher(vm, k, handler);
    }
  }
}
// 创建watcher的核心
function createWatcher(vm, exprOrFn, handler, options = {}) {
  if (typeof handler === "object") {
    options = handler; //保存用户传入的对象
    handler = handler.handler; //这个代表真正用户传入的函数
  }
  if (typeof handler === "string") {
    //   代表传入的是定义好的methods方法
    handler = vm[handler];
  }
  //   调用vm.$watch创建用户watcher
  return vm.$watch(exprOrFn, handler, options);
}
复制代码
```

initWatch初始化Watch对数组进行处理  createWatcher处理Watch的兼容性写法 包含字符串 函数 数组 以及对象 最后调用$watch 传入处理好的参数进行创建用户Watcher

#### 2.$watch

```javascript
//  src/state.js
import Watcher from "./observer/watcher";
Vue.prototype.$watch = function (exprOrFn, cb, options) {
  const vm = this;
  //  user: true 这里表示是一个用户watcher
  let watcher = new Watcher(vm, exprOrFn, cb, { ...options, user: true });
  // 如果有immediate属性 代表需要立即执行回调
  if (options.immediate) {
    cb(); //如果立刻执行
  }
};
复制代码
```

原型方法$watch 就是创建自定义 watch 的核心方法 把用户定义的 options 和 user:true 传给构造函数 Watcher

#### 3.Watcher 改造

```javascript
// src/observer/watcher.js

import { isObject } from "../util/index";
export default class Watcher {
  constructor(vm, exprOrFn, cb, options) {
    // this.vm = vm;
    // this.exprOrFn = exprOrFn;
    // this.cb = cb; //回调函数 比如在watcher更新之前可以执行beforeUpdate方法
    // this.options = options; //额外的选项 true代表渲染watcher
    // this.id = id++; // watcher的唯一标识
    // this.deps = []; //存放dep的容器
    // this.depsId = new Set(); //用来去重dep

    this.user = options.user; //标识用户watcher

    // 如果表达式是一个函数
    if (typeof exprOrFn === "function") {
      this.getter = exprOrFn;
    } else {
      this.getter = function () {
        //用户watcher传过来的可能是一个字符串   类似a.a.a.a.b
        let path = exprOrFn.split(".");
        let obj = vm;
        for (let i = 0; i < path.length; i++) {
          obj = obj[path[i]]; //vm.a.a.a.a.b
        }
        return obj;
      };
    }
    // 实例化就进行一次取值操作 进行依赖收集过程
    this.value = this.get();
  }
  //   get() {
  //     pushTarget(this); // 在调用方法之前先把当前watcher实例推到全局Dep.target上
  //     const res = this.getter.call(this.vm); //如果watcher是渲染watcher 那么就相当于执行  vm._update(vm._render()) 这个方法在render函数执行的时候会取值 从而实现依赖收集
  //     popTarget(); // 在调用方法之后把当前watcher实例从全局Dep.target移除
  //     return res;
  //   }
  //   把dep放到deps里面 同时保证同一个dep只被保存到watcher一次  同样的  同一个watcher也只会保存在dep一次
  //   addDep(dep) {
  //     let id = dep.id;
  //     if (!this.depsId.has(id)) {
  //       this.depsId.add(id);
  //       this.deps.push(dep);
  //       //   直接调用dep的addSub方法  把自己--watcher实例添加到dep的subs容器里面
  //       dep.addSub(this);
  //     }
  //   }
  //   这里简单的就执行以下get方法  之后涉及到计算属性就不一样了
  //   update() {
  //     // 计算属性依赖的值发生变化 只需要把dirty置为true  下次访问到了重新计算
  //     if (this.lazy) {
  //       this.dirty = true;
  //     }else{
  //       // 每次watcher进行更新的时候  可以让他们先缓存起来  之后再一起调用
  //       // 异步队列机制
  //       queueWatcher(this);
  //     }
  //   }
  //   depend(){
  //     // 计算属性的watcher存储了依赖项的dep
  //     let i=this.deps.length
  //     while(i--){
  //       this.deps[i].depend() //调用依赖项的dep去收集渲染watcher
  //     }
  //   }
  run() {
    const newVal = this.get(); //新值
    const oldVal = this.value; //老值
    this.value = newVal; //现在的新值将成为下一次变化的老值
    if (this.user) {
      // 如果两次的值不相同  或者值是引用类型 因为引用类型新老值是相等的 他们是指向同一引用地址
      if (newVal !== oldVal || isObject(newVal)) {
        this.cb.call(this.vm, newVal, oldVal);
      }
    } else {
      // 渲染watcher
      this.cb.call(this.vm);
    }
  }
}
复制代码
```

咱们主要关注非注释的地方  这里主要改造有两点

1.实例化的时候为了兼容用户 watch 的写法 会将传入的字符串写法转成 Vue 实例对应的值 并且调用 get 方法获取并保存一次旧值

2.run 方法判断如果是用户 watch 那么执行用户传入的回调函数 cb 并且把新值和旧值作为参数传入进去

#### 4.侦听属性的思维导图

![Vue2.0源码-侦听属性.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61fa92688a1e4ed2b000d362939a1133~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的 侦听属性原理已经完结 此篇相较之前的原理来说比较容易 其实计算属性和侦听属性全都是借助 Watcher 进行实现的 不清楚整个 Watcher 实现的可以看看小编这篇[手写 Vue2.0 源码（四）-渲染更新原理](https://juejin.cn/post/6938221715281575973) 下一篇是计算属性原理 会和侦听属性进行对比 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言



此篇主要手写 Vue2.0 源码-**计算属性**

上一篇咱们主要介绍了 Vue [侦听属性原理](https://juejin.cn/post/6954925963226382367) 知道了用户定义的 watch 如何被创建 此篇我们介绍他的兄弟-计算属性 主要特性是如果计算属性依赖的值不发生变化 页面更新的时候不会重新计算 计算结果会被缓存 可以用此 api 来优化性能

**适用人群：**

1.想要深入理解 vue 源码更好的进行日常业务开发

2.想要在简历写上精通 vue 框架源码（再也不怕面试官的连环夺命问 哈哈）

3.没时间去看官方源码或者初看源码觉得难以理解的同学

------

### 正文

```html
<script>
  // Vue实例化
  let vm = new Vue({
    el: "#app",
    data() {
      return {
        aa: 1,
        bb: 2,
        cc: 3,
      };
    },
    // render(h) {
    //   return h('div',{id:'a'},'hello')
    // },
    template: `<div id="a">hello 这是我自己写的Vue{{computedName}}{{cc}}</div>`,
    computed: {
      computedName() {
        return this.aa + this.bb;
      },
    },
  });
  // 当我们每一次改变数据的时候  渲染watcher都会执行一次 这个是影响性能的
  setTimeout(() => {
    vm.cc = 4;
  }, 2000);
  console.log(vm);
</script>
复制代码
```

上述例子 就是计算属性的基础用法 我们在两秒之后改变了模板里面的 cc 但是计算属性依赖的 aa 和 bb 都没变化 所以计算属性不会重新计算 还是保留的上次计算结果

#### 1.计算属性的初始化

```javascript
// src/state.js

function initComputed(vm) {
  const computed = vm.$options.computed;

  const watchers = (vm._computedWatchers = {}); //用来存放计算watcher

  for (let k in computed) {
    const userDef = computed[k]; //获取用户定义的计算属性
    const getter = typeof userDef === "function" ? userDef : userDef.get; //创建计算属性watcher使用
    // 创建计算watcher  lazy设置为true
    watchers[k] = new Watcher(vm, getter, () => {}, { lazy: true });
    defineComputed(vm, k, userDef);
  }
}
复制代码
```

计算属性可以写成一个函数也可以写成一个对象 对象的形式 get 属性就代表的是计算属性依赖的值 set 代表修改计算属性的依赖项的值 我们主要关心 get 属性 然后类似侦听属性 我们把 lazy:true 传给构造函数 Watcher 用来创建计算属性 Watcher 那么 defineComputed 是什么意思呢

> 思考？ 计算属性是可以缓存计算结果的 我们应该怎么做？

#### 2.对计算属性进行属性劫持

```javascript
//  src/state.js

// 定义普通对象用来劫持计算属性
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: () => {},
  set: () => {},
};

// 重新定义计算属性  对get和set劫持
function defineComputed(target, key, userDef) {
  if (typeof userDef === "function") {
    // 如果是一个函数  需要手动赋值到get上
    sharedPropertyDefinition.get = createComputedGetter(key);
  } else {
    sharedPropertyDefinition.get = createComputedGetter(key);
    sharedPropertyDefinition.set = userDef.set;
  }
  //   利用Object.defineProperty来对计算属性的get和set进行劫持
  Object.defineProperty(target, key, sharedPropertyDefinition);
}

// 重写计算属性的get方法 来判断是否需要进行重新计算
function createComputedGetter(key) {
  return function () {
    const watcher = this._computedWatchers[key]; //获取对应的计算属性watcher
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate(); //计算属性取值的时候 如果是脏的  需要重新求值
      }
      return watcher.value;
    }
  };
}
复制代码
```

defineComputed 方法主要是重新定义计算属性 其实最主要的是劫持 get 方法 也就是计算属性依赖的值 为啥要劫持呢 因为我们需要根据依赖值是否发生变化来判断计算属性是否需要重新计算

createComputedGetter 方法就是判断计算属性依赖的值是否变化的核心了 我们在计算属性创建的 Watcher 增加 dirty 标志位 如果标志变为 true 代表需要调用 watcher.evaluate 来进行重新计算了

#### 3.Watcher 改造

```javascript
// src/observer/watcher.js

// import { pushTarget, popTarget } from "./dep";
// import { queueWatcher } from "./scheduler";
// import {isObject} from '../util/index'
// // 全局变量id  每次new Watcher都会自增
// let id = 0;

export default class Watcher {
  constructor(vm, exprOrFn, cb, options) {
    // this.vm = vm;
    // this.exprOrFn = exprOrFn;
    // this.cb = cb; //回调函数 比如在watcher更新之前可以执行beforeUpdate方法
    // this.options = options; //额外的选项 true代表渲染watcher
    // this.id = id++; // watcher的唯一标识
    // this.deps = []; //存放dep的容器
    // this.depsId = new Set(); //用来去重dep
    // this.user = options.user; //标识用户watcher
    this.lazy = options.lazy; //标识计算属性watcher
    this.dirty = this.lazy; //dirty可变  表示计算watcher是否需要重新计算 默认值是true

    // 如果表达式是一个函数
    // if (typeof exprOrFn === "function") {
    //   this.getter = exprOrFn;
    // } else {
    //   this.getter = function () {
    //     //用户watcher传过来的可能是一个字符串   类似a.a.a.a.b
    //     let path = exprOrFn.split(".");
    //     let obj = vm;
    //     for (let i = 0; i < path.length; i++) {
    //       obj = obj[path[i]]; //vm.a.a.a.a.b
    //     }
    //     return obj;
    //   };
    // }
    // 非计算属性实例化就会默认调用get方法 进行取值  保留结果 计算属性实例化的时候不会去调用get
    this.value = this.lazy ? undefined : this.get();
  }
  get() {
    pushTarget(this); // 在调用方法之前先把当前watcher实例推到全局Dep.target上
    const res = this.getter.call(this.vm); //计算属性在这里执行用户定义的get函数 访问计算属性的依赖项 从而把自身计算Watcher添加到依赖项dep里面收集起来
    popTarget(); // 在调用方法之后把当前watcher实例从全局Dep.target移除
    return res;
  }
  //   把dep放到deps里面 同时保证同一个dep只被保存到watcher一次  同样的  同一个watcher也只会保存在dep一次
  //   addDep(dep) {
  //     let id = dep.id;
  //     if (!this.depsId.has(id)) {
  //       this.depsId.add(id);
  //       this.deps.push(dep);
  //       //   直接调用dep的addSub方法  把自己--watcher实例添加到dep的subs容器里面
  //       dep.addSub(this);
  //     }
  //   }
  //   这里简单的就执行以下get方法  之后涉及到计算属性就不一样了
  update() {
    // 计算属性依赖的值发生变化 只需要把dirty置为true  下次访问到了重新计算
    if (this.lazy) {
      this.dirty = true;
    } else {
      // 每次watcher进行更新的时候  可以让他们先缓存起来  之后再一起调用
      // 异步队列机制
      queueWatcher(this);
    }
  }
  //   计算属性重新进行计算 并且计算完成把dirty置为false
  evaluate() {
    this.value = this.get();
    this.dirty = false;
  }
  depend() {
    // 计算属性的watcher存储了依赖项的dep
    let i = this.deps.length;
    while (i--) {
      this.deps[i].depend(); //调用依赖项的dep去收集渲染watcher
    }
  }
  //   run() {
  //     const newVal = this.get(); //新值
  //     const oldVal = this.value; //老值
  //     this.value = newVal; //跟着之后  老值就成为了现在的值
  //     if (this.user) {
  //       if(newVal!==oldVal||isObject(newVal)){
  //         this.cb.call(this.vm, newVal, oldVal);
  //       }
  //     } else {
  //       // 渲染watcher
  //       this.cb.call(this.vm);
  //     }
  //   }
}
复制代码
```

我们主要看没被注释的代码 这里主要改造有四点

1.实例化的时候如果是计算属性 不会去调用 get 方法访问值进行依赖收集

2.update 方法只是把计算 watcher 的 dirty 标识为 true 只有当下次访问到了计算属性的时候才会重新计算

3.新增 evaluate 方法专门用于计算属性重新计算

4.新增 depend 方法 让计算属性的依赖值收集外层 watcher--这个方法非常重要 我们接下来分析

#### 4.外层 Watcher 的依赖收集

```javascript
// src/state.js

function createComputedGetter(key) {
//   return function () {
//     const watcher = this._computedWatchers[key]; //获取对应的计算属性watcher
//     if (watcher) {
//       if (watcher.dirty) {
//         watcher.evaluate(); //计算属性取值的时候 如果是脏的  需要重新求值
        if (Dep.target) {
    // 如果Dep还存在target 这个时候一般为渲染watcher 计算属性依赖的数据也需要收集
          watcher.depend()
        }
//       }
//       return watcher.value;
//     }
//   };
// }

复制代码
```

这里就体现了 watcher.depend 方法的重要性了 我们试想一下 当我们计算属性依赖的值发生了改变 这时候 watcher 的 dirty 为 true 下次访问计算属性 他确实也重新计算了 但是 我们从头到尾都没有触发视图更新 也就是数据改变了 视图没有重新渲染

这是为什么呢？

因为模板里面只有计算属性 而计算属性的依赖值的 dep 里面只收集了计算 watcher 的依赖 自身变化也只是通知了计算 watcher 调用 update 把 dirty 置为 true 所以我们要想个办法把计算属性的依赖项也添加渲染 watcher 的依赖 让自身变化之后首先通知计算 watcher 进行重新计算 然后通知渲染 watcher 进行视图更新

怎么做呢？我们来看看下面的代码就清楚了

```javascript
// src/observer/dep.js

// 默认Dep.target为null
Dep.target = null;
// 栈结构用来存watcher
const targetStack = [];

export function pushTarget(watcher) {
  targetStack.push(watcher);
  Dep.target = watcher; // Dep.target指向当前watcher
}
export function popTarget() {
  targetStack.pop(); // 当前watcher出栈 拿到上一个watcher
  Dep.target = targetStack[targetStack.length - 1];
}
复制代码
```

可见最初设计存放 watcher 的容器就是一个栈结构 因为整个 Vue 生命周期的过程中会存在很多的 watcher 比如渲染 watcher 计算 watcher 侦听 watcher 等 而每个 watcher 在调用了自身的 get 方法前后会分别调用 pushTarget 入栈和 popTarget 出栈 这样子当计算属性重新计算之后就立马会出栈 那么外层的 watcher 就会成为新的 Dep.target 我们使用 watcher.depend 方法让计算属性依赖的值收集一遍外层的渲染 watcher 这样子当计算属性依赖的值改变了既可以重新计算又可以刷新视图

> 对于渲染更新不了解的同学建议看看小编这篇 [手写 Vue2.0 源码（四）-渲染更新原理](https://juejin.cn/post/6938221715281575973)

#### 5.计算属性的思维导图

![Vue2.0源码-计算属性.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2133fd4375c445a6adc8d7426c0bc2ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的 计算属性原理已经完结 和侦听属性还是有很大区别的 计算属性一般用在需要对依赖项进行计算并且可以缓存下来 当依赖项变化会自动执行计算属性的逻辑 一般用在模板里面较多 而侦听属性用法是对某个响应式的值进行观察（也可以观察计算属性的值） 一旦变化之后就可以执行自己定义的方法 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言



此篇主要手写 Vue2.0 源码-**全局 api 原理**

上一篇咱们主要介绍了 Vue [计算属性原理](https://link.juejin.cn?target=) 知道了计算属性缓存的特点是怎么实现的 到目前为止整个 Vue 源码核心内容 咱们已经基本手写了一遍 那么此篇来梳理下 Vue 的全局 api

**适用人群：**

1.想要深入理解 vue 源码更好的进行日常业务开发

2.想要在简历写上精通 vue 框架源码（再也不怕面试官的连环夺命问 哈哈）

3.没时间去看官方源码或者初看源码觉得难以理解的同学

------

#### 1 Vue.util

```javascript
// src/global-api/index.js

// exposed util methods.
// NOTE: these are not considered part of the public API - avoid relying on
// them unless you are aware of the risk.
Vue.util = {
  warn,
  extend,
  mergeOptions,
  defineReactive,
};
复制代码
```

Vue.util 是 Vue 内部的工具方法 不推荐业务组件去使用 因为可能随着版本发生变动 如果咱们不开发第三方 Vue 插件确实使用会比较少

#### 2 Vue.set / Vue.delete

```javascript
export function set(target: Array<any> | Object, key: any, val: any): any {
  // 如果是数组 直接调用我们重写的splice方法 可以刷新视图
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);
    target.splice(key, 1, val);
    return val;
  }
  // 如果是对象本身的属性，则直接添加即可
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val;
  }
  const ob = (target: any).__ob__;

  // 如果对象本身就不是响应式 不需要将其定义成响应式属性
  if (!ob) {
    target[key] = val;
    return val;
  }
  // 利用defineReactive   实际就是Object.defineProperty 将新增的属性定义成响应式的
  defineReactive(ob.value, key, val);
  ob.dep.notify(); // 通知视图更新
  return val;
}
复制代码
export function del(target: Array<any> | Object, key: any) {
  // 如果是数组依旧调用splice方法
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1);
    return;
  }
  const ob = (target: any).__ob__;
  // 如果对象本身就没有这个属性 什么都不做
  if (!hasOwn(target, key)) {
    return;
  }
  // 直接使用delete  删除这个属性
  delete target[key];
  //   如果对象本身就不是响应式 直接返回
  if (!ob) {
    return;
  }
  ob.dep.notify(); //通知视图更新
}
复制代码
```

这两个 api 其实在实际业务场景使用还是很多的 set 方法用来新增响应式数据 delete 方法用来删除响应式数据 因为 Vue 整个响应式过程是依赖 Object.defineProperty 这一底层 api 的 但是这个 api 只能对当前已经声明过的对象属性进行劫持 所以新增的属性不是响应式数据 另外直接修改数组下标也不会引发视图更新 这个是考虑到性能原因 所以我们需要使用$set 和$delete 来进行操作 对响应式原理不熟悉的可以看[手写 Vue2.0 源码（一）-响应式数据原理](https://juejin.cn/post/6935344605424517128)

#### 3 Vue.nextTick

```javascript
let callbacks = []; //回调函数
let pending = false;
function flushCallbacks() {
  pending = false; //把标志还原为false
  // 依次执行回调
  for (let i = 0; i < callbacks.length; i++) {
    callbacks[i]();
  }
}
let timerFunc; //先采用微任务并按照优先级优雅降级的方式实现异步刷新
if (typeof Promise !== "undefined") {
  // 如果支持promise
  const p = Promise.resolve();
  timerFunc = () => {
    p.then(flushCallbacks);
  };
} else if (typeof MutationObserver !== "undefined") {
  // MutationObserver 主要是监听dom变化 也是一个异步方法
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true,
  });
  timerFunc = () => {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
} else if (typeof setImmediate !== "undefined") {
  // 如果前面都不支持 判断setImmediate
  timerFunc = () => {
    setImmediate(flushCallbacks);
  };
} else {
  // 最后降级采用setTimeout
  timerFunc = () => {
    setTimeout(flushCallbacks, 0);
  };
}

export function nextTick(cb) {
  // 除了渲染watcher  还有用户自己手动调用的nextTick 一起被收集到数组
  callbacks.push(cb);
  if (!pending) {
    // 如果多次调用nextTick  只会执行一次异步 等异步队列清空之后再把标志变为false
    pending = true;
    timerFunc();
  }
}
复制代码
```

nextTick 是 Vue 实现异步更新的核心 此 api 在实际业务使用频次也很高 一般用作在数据改变之后立马要获取 dom 节点相关的属性 那么就可以把这样的方法放在 nextTick 中去实现 异步更新原理可以看[手写 Vue2.0 源码（五）-异步更新原理](https://juejin.cn/post/6939704519668432910)

#### 4 Vue.observable

```javascript
Vue.observable = <T>(obj: T): T => {
  observe(obj);
  return obj;
};
复制代码
```

核心就是调用 observe 方法将传入的数据变成响应式对象 可用于制造全局变量在组件共享数据 具体 observe 方法可以看[响应式数据原理-对象的数据劫持](https://juejin.cn/post/6935344605424517128#heading-3)

#### 5 Vue.options

```javascript
Vue.options = Object.create(null);
ASSET_TYPES.forEach((type) => {
  Vue.options[type + "s"] = Object.create(null);
});

// this is used to identify the "base" constructor to extend all plain-object
// components with in Weex's multi-instance scenarios.
Vue.options._base = Vue;

extend(Vue.options.components, builtInComponents); //内置组件
复制代码
```

Vue.options 是存放组件 指令和过滤器的容器 并且 Vue.options._base 指向 Vue 构造函数

#### 6 Vue.use

```javascript
Vue.use = function (plugin: Function | Object) {
  const installedPlugins =
    this._installedPlugins || (this._installedPlugins = []);
  if (installedPlugins.indexOf(plugin) > -1) {
    // 如果安装过这个插件直接返回
    return this;
  }

  const args = toArray(arguments, 1); // 获取参数
  args.unshift(this); //在参数中增加Vue构造函数

  if (typeof plugin.install === "function") {
    plugin.install.apply(plugin, args); // 执行install方法
  } else if (typeof plugin === "function") {
    plugin.apply(null, args); // 没有install方法直接把传入的插件执行
  }
  // 记录安装的插件
  installedPlugins.push(plugin);
  return this;
};
复制代码
```

Vue.use 主要用于插件的注册 调用插件的 install 方法 并且把自身 Vue 传到插件的 install 方法 这样可以避免第三方插件强依赖 Vue

#### 7 Vue.mixin

```javascript
export function initMixin(Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    this.options = mergeOptions(this.options, mixin); //只要调用mergeOptions来合并选项
    return this;
  };
}

/**
 * Merge two option objects into a new one.
 * Core utility used in both instantiation and inheritance.
 */
export function mergeOptions(
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (!child._base) {
    //   这个代表是组件  需要先把自己定义的extends和mixins与父级属性进行合并
    if (child.extends) {
      parent = mergeOptions(parent, child.extends, vm);
    }
    if (child.mixins) {
      for (let i = 0, l = child.mixins.length; i < l; i++) {
        parent = mergeOptions(parent, child.mixins[i], vm);
      }
    }
  }

  // 把自己的和父亲的属性进行合并
  const options = {};
  let key;
  for (key in parent) {
    mergeField(key);
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key);
    }
  }
  function mergeField(key) {
    //真正合并字段的方法
    const strat = strats[key] || defaultStrat; //strats代表合并策略 会优先查找对应的合并策略 找不到就用默认的合并策略
    options[key] = strat(parent[key], child[key], vm, key);
  }
  return options;
}
复制代码
```

Vue.mixin 是全局混入方法 一般用作提取全局的公共方法和属性 想深入了解这块的可以看[手写 Vue2.0 源码（七）-Mixin 混入原理 ](https://juejin.cn/post/6951671158198501383)

#### 8 Vue.extend

```javascript
Vue.extend = function (extendOptions: Object): Function {
  const Sub = function VueComponent(options) {
    // 创建子类的构造函数 并且调用初始化方法
    this._init(options);
  };
  Sub.prototype = Object.create(Super.prototype); // 子类原型指向父类
  Sub.prototype.constructor = Sub; //constructor指向自己
  Sub.options = mergeOptions(
    //合并自己的options和父类的options
    Super.options,
    extendOptions
  );
  return Sub;
};
复制代码
```

Vue.extend 被称为组件构造器 Vue 的组件创建就是依赖于此 api 其实就是利用原型继承的方式创建继承自 Vue 的子类 对组件初始化和渲染感兴趣的可以看[手写 Vue2.0 源码（八）-组件原理](https://juejin.cn/post/6954173708344770591)

#### 9 组件、指令、过滤器

```javascript
export function initAssetRegisters(Vue: GlobalAPI) {
  var ASSET_TYPES = ["component", "directive", "filter"];
  /**
   * Create asset registration methods.
   */
  ASSET_TYPES.forEach((type) => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + "s"][id];
      } else {
        if (type === "component" && isPlainObject(definition)) {
          definition.name = definition.name || id;
          definition = this.options._base.extend(definition);
        }
        if (type === "directive" && typeof definition === "function") {
          definition = { bind: definition, update: definition };
        }
        this.options[type + "s"][id] = definition; //把组件  指令  过滤器 放到Vue.options中
        return definition;
      }
    };
  });
}
复制代码
```

定义 Vue.component Vue.directive Vue.filter 三大 api 并且格式化用户传入内容 最后把结果放到 Vue.options 中

#### 10 全局 api 思维导图

![Vue2.0源码-全局api原理.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c609acf748ae45b68604f616185792a4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 小结

至此 Vue 的 全局 api 原理已经完结 基本上很多代码咱们之前 Vue 系列的原理都有写过 很多核心 api 大家在日常开发过程中使用的比较频繁 同时也是面试的常见考点 大家可以看着思维导图自己动手写一遍核心代码哈 遇到不懂或者有争议的地方欢迎评论留言

### 简单

#### 1 MVC 和 MVVM 区别

##### MVC

MVC 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范

- Model（模型）：是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据
- View（视图）：是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的
- Controller（控制器）：是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据

![mvc.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e4d22e916014ee7abb10e4b350e5583~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

MVC 的思想：一句话描述就是 Controller 负责将 Model 的数据用 View 显示出来，换句话说就是在 Controller 里面把 Model 的数据赋值给 View。

##### MVVM

MVVM 新增了 VM 类

- ViewModel 层：做了两件事达到了数据的双向绑定 一是将【模型】转化成【视图】，即将后端传递的数据转化成所看到的页面。实现的方式是：数据绑定。二是将【视图】转化成【模型】，即将所看到的页面转化成后端的数据。实现的方式是：DOM 事件监听。

![mvvm.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32c0545686454396a7e39b86b644fe73~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

MVVM 与 MVC 最大的区别就是：它实现了 View 和 Model 的自动同步，也就是当 Model 的属性改变时，我们不用再自己手动操作 Dom 元素，来改变 View 的显示，而是改变属性后该属性对应 View 层显示会自动改变（对应Vue数据驱动的思想）

整体看来，MVVM 比 MVC 精简很多，不仅简化了业务与界面的依赖，还解决了数据频繁更新的问题，不用再用选择器操作 DOM 元素。因为在 MVVM 中，View 不知道 Model 的存在，Model 和 ViewModel 也观察不到 View，这种低耦合模式提高代码的可重用性

> 注意：Vue 并没有完全遵循 MVVM 的思想 这一点官网自己也有说明

![vue-mvvm.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0290c7c656b449718db920a5456d0f45~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

那么问题来了 为什么官方要说 Vue 没有完全遵循 MVVM 思想呢？

> - 严格的 MVVM 要求 View 不能和 Model 直接通信，而 Vue 提供了$refs 这个属性，让 Model 可以直接操作 View，违反了这一规定，所以说 Vue 没有完全遵循 MVVM。

#### 2 为什么 data 是一个函数

组件中的 data 写成一个函数，数据以函数返回值形式定义，这样每复用一次组件，就会返回一份新的 data，类似于给每个组件实例创建一个私有的数据空间，让各个组件实例维护各自的数据。而单纯的写成对象形式，就使得所有组件实例共用了一份 data，就会造成一个变了全都会变的结果

#### 3 Vue 组件通讯有哪几种方式

1. props 和$emit 父组件向子组件传递数据是通过 prop 传递的，子组件传递数据给父组件是通过$emit 触发事件来做到的
2. $parent,$children 获取当前组件的父组件和当前组件的子组件
3. $attrs 和$listeners A->B->C。Vue 2.4 开始提供了$attrs 和$listeners 来解决这个问题
4. 父组件中通过 provide 来提供变量，然后在子组件中通过 inject 来注入变量。(官方不推荐在实际业务中使用，但是写组件库时很常用)
5. $refs 获取组件实例
6. eventBus 兄弟组件数据传递 这种情况下可以使用事件总线的方式
7. vuex 状态管理

#### 4 Vue 的生命周期方法有哪些 一般在哪一步发请求

**beforeCreate** 在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。在当前阶段 data、methods、computed 以及 watch 上的数据和方法都不能被访问

**created** 实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。这里没有$el,如果非要想与 Dom 进行交互，可以通过 vm.$nextTick 来访问 Dom

**beforeMount** 在挂载开始之前被调用：相关的 render 函数首次被调用。

**mounted** 在挂载完成后发生，在当前阶段，真实的 Dom 挂载完毕，数据完成双向绑定，可以访问到 Dom 节点

**beforeUpdate** 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁（patch）之前。可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程

**updated** 发生在更新完成之后，当前阶段组件 Dom 已完成更新。要注意的是避免在此期间更改数据，因为这可能会导致无限循环的更新，该钩子在服务器端渲染期间不被调用。

**beforeDestroy** 实例销毁之前调用。在这一步，实例仍然完全可用。我们可以在这时进行善后收尾工作，比如清除计时器。

**destroyed** Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。 该钩子在服务器端渲染期间不被调用。

**activated** keep-alive 专属，组件被激活时调用

**deactivated** keep-alive 专属，组件被销毁时调用

> 异步请求在哪一步发起？

可以在钩子函数 created、beforeMount、mounted 中进行异步请求，因为在这三个钩子函数中，data 已经创建，可以将服务端端返回的数据进行赋值。

如果异步请求不需要依赖 Dom 推荐在 created 钩子函数中调用异步请求，因为在 created 钩子函数中调用异步请求有以下优点：

- 能更快获取到服务端数据，减少页面  loading 时间；
- ssr  不支持 beforeMount 、mounted 钩子函数，所以放在 created 中有助于一致性；

#### 5 v-if 和 v-show 的区别

v-if 在编译过程中会被转化成三元表达式,条件不满足时不渲染此节点。

v-show 会被编译成指令，条件不满足时控制样式将对应节点隐藏 （display:none）

**使用场景**

v-if 适用于在运行时很少改变条件，不需要频繁切换条件的场景

v-show 适用于需要非常频繁切换条件的场景

> 扩展补充：display:none、visibility:hidden 和 opacity:0 之间的区别？

![display.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/618888ae9baa4c3096479b1f61bb37f3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 6 说说 vue 内置指令

![内置指令.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b46ec8b051246858211c4c7ec129fb3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

#### 7 怎样理解 Vue 的单向数据流

数据总是从父组件传到子组件，子组件没有权利修改父组件传过来的数据，只能请求父组件对原始数据进行修改。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

> 注意：在子组件直接用 v-model 绑定父组件传过来的 prop 这样是不规范的写法 开发环境会报警告

如果实在要改变父组件的 prop 值 可以再 data 里面定义一个变量 并用 prop 的值初始化它 之后用$emit 通知父组件去修改

#### 8 computed 和 watch 的区别和运用的场景

computed 是计算属性，依赖其他属性计算值，并且 computed 的值有缓存，只有当计算值变化才会返回内容，它可以设置 getter 和 setter。

watch 监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作。

计算属性一般用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；而侦听属性适用于观测某个值的变化去完成一段复杂的业务逻辑

计算属性原理详解 [传送门](https://juejin.cn/post/6956407362085191717)

侦听属性原理详解 [传送门](https://juejin.cn/post/6954925963226382367)

#### 9 v-if 与 v-for 为什么不建议一起使用

v-for 和 v-if 不要在同一个标签中使用,因为解析时先解析 v-for 再解析 v-if。如果遇到需要同时使用时可以考虑写成计算属性的方式。

------

### 中等

#### 10 Vue2.0 响应式数据的原理

整体思路是数据劫持+观察者模式

对象内部通过 defineReactive 方法，使用 Object.defineProperty 将属性进行劫持（只会劫持已经存在的属性），数组则是通过重写数组方法来实现。当页面使用对应属性时，每个属性都拥有自己的 dep 属性，存放他所依赖的 watcher（依赖收集），当属性变化后会通知自己对应的 watcher 去更新(派发更新)。

相关代码如下

```javascript
class Observer {
  // 观测值
  constructor(value) {
    this.walk(value);
  }
  walk(data) {
    // 对象上的所有属性依次进行观测
    let keys = Object.keys(data);
    for (let i = 0; i < keys.length; i++) {
      let key = keys[i];
      let value = data[key];
      defineReactive(data, key, value);
    }
  }
}
// Object.defineProperty数据劫持核心 兼容性在ie9以及以上
function defineReactive(data, key, value) {
  observe(value); // 递归关键
  // --如果value还是一个对象会继续走一遍odefineReactive 层层遍历一直到value不是对象才停止
  //   思考？如果Vue数据嵌套层级过深 >>性能会受影响
  Object.defineProperty(data, key, {
    get() {
      console.log("获取值");

      //需要做依赖收集过程 这里代码没写出来
      return value;
    },
    set(newValue) {
      if (newValue === value) return;
      console.log("设置值");
      //需要做派发更新过程 这里代码没写出来
      value = newValue;
    },
  });
}
export function observe(value) {
  // 如果传过来的是对象或者数组 进行属性劫持
  if (
    Object.prototype.toString.call(value) === "[object Object]" ||
    Array.isArray(value)
  ) {
    return new Observer(value);
  }
}
复制代码
```

响应式数据原理详解 [传送门](https://juejin.cn/post/6935344605424517128)

#### 11 Vue 如何检测数组变化

数组考虑性能原因没有用 defineProperty 对数组的每一项进行拦截，而是选择对 7 种数组（push,shift,pop,splice,unshift,sort,reverse）方法进行重写(AOP 切片思想)

所以在 Vue 中修改数组的索引和长度是无法监控到的。需要通过以上 7 种变异方法修改数组才会触发数组对应的 watcher 进行更新

相关代码如下

```javascript
// src/obserber/array.js
// 先保留数组原型
const arrayProto = Array.prototype;
// 然后将arrayMethods继承自数组原型
// 这里是面向切片编程思想（AOP）--不破坏封装的前提下，动态的扩展功能
export const arrayMethods = Object.create(arrayProto);
let methodsToPatch = [
  "push",
  "pop",
  "shift",
  "unshift",
  "splice",
  "reverse",
  "sort",
];
methodsToPatch.forEach((method) => {
  arrayMethods[method] = function (...args) {
    //   这里保留原型方法的执行结果
    const result = arrayProto[method].apply(this, args);
    // 这句话是关键
    // this代表的就是数据本身 比如数据是{a:[1,2,3]} 那么我们使用a.push(4)  this就是a  ob就是a.__ob__ 这个属性就是上段代码增加的 代表的是该数据已经被响应式观察过了指向Observer实例
    const ob = this.__ob__;

    // 这里的标志就是代表数组有新增操作
    let inserted;
    switch (method) {
      case "push":
      case "unshift":
        inserted = args;
        break;
      case "splice":
        inserted = args.slice(2);
      default:
        break;
    }
    // 如果有新增的元素 inserted是一个数组 调用Observer实例的observeArray对数组每一项进行观测
    if (inserted) ob.observeArray(inserted);
    // 之后咱们还可以在这里检测到数组改变了之后从而触发视图更新的操作--后续源码会揭晓
    return result;
  };
});
复制代码
```

数组的观测原理详解 [传送门](https://juejin.cn/post/6935344605424517128#heading-4)

#### 12 vue3.0 用过吗 了解多少

- 响应式原理的改变 Vue3.x 使用 Proxy 取代 Vue2.x 版本的 Object.defineProperty
- 组件选项声明方式 Vue3.x 使用 Composition API setup 是 Vue3.x 新增的一个选项， 他是组件内使用 Composition API 的入口。
- 模板语法变化 slot 具名插槽语法 自定义指令 v-model 升级
- 其它方面的更改 Suspense 支持 Fragment（多个根节点）和 Protal（在 dom 其他部分渲染组建内容）组件，针对一些特殊的场景做了处理。 基于 treeshaking 优化，提供了更多的内置功能。

Vue3.0 新特性以及使用经验总结 [传送门](https://juejin.cn/post/6940454764421316644)

#### 13 Vue3.0 和 2.0 的响应式原理区别

Vue3.x 改用 Proxy 替代 Object.defineProperty。因为 Proxy 可以直接监听对象和数组的变化，并且有多达 13 种拦截方法。

相关代码如下

```javascript
import { mutableHandlers } from "./baseHandlers"; // 代理相关逻辑
import { isObject } from "./util"; // 工具方法

export function reactive(target) {
  // 根据不同参数创建不同响应式对象
  return createReactiveObject(target, mutableHandlers);
}
function createReactiveObject(target, baseHandler) {
  if (!isObject(target)) {
    return target;
  }
  const observed = new Proxy(target, baseHandler);
  return observed;
}

const get = createGetter();
const set = createSetter();

function createGetter() {
  return function get(target, key, receiver) {
    // 对获取的值进行放射
    const res = Reflect.get(target, key, receiver);
    console.log("属性获取", key);
    if (isObject(res)) {
      // 如果获取的值是对象类型，则返回当前对象的代理对象
      return reactive(res);
    }
    return res;
  };
}
function createSetter() {
  return function set(target, key, value, receiver) {
    const oldValue = target[key];
    const hadKey = hasOwn(target, key);
    const result = Reflect.set(target, key, value, receiver);
    if (!hadKey) {
      console.log("属性新增", key, value);
    } else if (hasChanged(value, oldValue)) {
      console.log("属性值被修改", key, value);
    }
    return result;
  };
}
export const mutableHandlers = {
  get, // 当获取属性时调用此方法
  set, // 当修改属性时调用此方法
};
复制代码
```

#### 14 Vue 的父子组件生命周期钩子函数执行顺序

- 加载渲染过程

父 beforeCreate->父 created->父 beforeMount->子 beforeCreate->子 created->子 beforeMount->子 mounted->父 mounted

- 子组件更新过程

父 beforeUpdate->子 beforeUpdate->子 updated->父 updated

- 父组件更新过程

父 beforeUpdate->父 updated

- 销毁过程

父 beforeDestroy->子 beforeDestroy->子 destroyed->父 destroyed

#### 15 虚拟 DOM 是什么 有什么优缺点

由于在浏览器中操作 DOM 是很昂贵的。频繁的操作 DOM，会产生一定的性能问题。这就是虚拟 Dom 的产生原因。Vue2 的 Virtual DOM 借鉴了开源库 snabbdom 的实现。Virtual DOM 本质就是用一个原生的 JS 对象去描述一个 DOM 节点，是对真实 DOM 的一层抽象。

**优点：**

1. 保证性能下限： 框架的虚拟 DOM 需要适配任何上层 API 可能产生的操作，它的一些 DOM 操作的实现必须是普适的，所以它的性能并不是最优的；但是比起粗暴的 DOM 操作性能要好很多，因此框架的虚拟 DOM 至少可以保证在你不需要手动优化的情况下，依然可以提供还不错的性能，即保证性能的下限；
2. 无需手动操作 DOM： 我们不再需要手动去操作 DOM，只需要写好 View-Model 的代码逻辑，框架会根据虚拟 DOM 和 数据双向绑定，帮我们以可预期的方式更新视图，极大提高我们的开发效率；
3. 跨平台： 虚拟 DOM 本质上是 JavaScript 对象,而 DOM 与平台强相关，相比之下虚拟 DOM 可以进行更方便地跨平台操作，例如服务器渲染、weex 开发等等。

**缺点:**

1. 无法进行极致优化： 虽然虚拟 DOM + 合理的优化，足以应对绝大部分应用的性能需求，但在一些性能要求极高的应用中虚拟 DOM 无法进行针对性的极致优化。
2. 首次渲染大量 DOM 时，由于多了一层虚拟 DOM 的计算，会比 innerHTML 插入慢。

#### 16 v-model 原理

v-model 只是语法糖而已

v-model 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：

- text 和 textarea 元素使用 value property 和 input 事件；
- checkbox 和 radio 使用 checked property 和 change 事件；
- select 字段将 value 作为 prop 并将 change 作为事件。

> 注意:对于需要使用输入法 (如中文、日文、韩文等) 的语言，你会发现 v-model 不会在输入法组合文字过程中得到更新。

在普通标签上

```javascript
    <input v-model="sth" />  //这一行等于下一行
    <input v-bind:value="sth" v-on:input="sth = $event.target.value" />
复制代码
```

在组件上

```html
<currency-input v-model="price"></currentcy-input>
<!--上行代码是下行的语法糖
 <currency-input :value="price" @input="price = arguments[0]"></currency-input>
-->

<!-- 子组件定义 -->
Vue.component('currency-input', {
 template: `
  <span>
   <input
    ref="input"
    :value="value"
    @input="$emit('input', $event.target.value)"
   >
  </span>
 `,
 props: ['value'],
})

复制代码
```

#### 17 v-for 为什么要加 key

如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。key 是为 Vue 中 vnode 的唯一标记，通过这个 key，我们的 diff 操作可以更准确、更快速

**更准确**：因为带 key 就不是就地复用了，在 sameNode 函数 a.key === b.key 对比中可以避免就地复用的情况。所以会更加准确。

**更快速**：利用 key 的唯一性生成 map 对象来获取对应节点，比遍历方式更快

相关代码如下

```javascript
// 判断两个vnode的标签和key是否相同 如果相同 就可以认为是同一节点就地复用
function isSameVnode(oldVnode, newVnode) {
  return oldVnode.tag === newVnode.tag && oldVnode.key === newVnode.key;
}

// 根据key来创建老的儿子的index映射表  类似 {'a':0,'b':1} 代表key为'a'的节点在第一个位置 key为'b'的节点在第二个位置
function makeIndexByKey(children) {
  let map = {};
  children.forEach((item, index) => {
    map[item.key] = index;
  });
  return map;
}
// 生成的映射表
let map = makeIndexByKey(oldCh);
复制代码
```

diff 算法详解 [传送门](https://juejin.cn/post/6953433215218483236)

#### 18 Vue 事件绑定原理

原生事件绑定是通过 addEventListener 绑定给真实元素的，组件事件绑定是通过 Vue 自定义的$on 实现的。如果要在组件上使用原生事件，需要加.native 修饰符，这样就相当于在父组件中把子组件当做普通 html 标签，然后加上原生事件。

$on、$emit 是基于发布订阅模式的，维护一个事件中心，on 的时候将事件按名称存在事件中心里，称之为订阅者，然后 emit 将对应的事件进行发布，去执行事件中心里的对应的监听器

手写发布订阅原理 [传送门](https://juejin.cn/post/6844904153437700103#heading-2)

#### 19 vue-router 路由钩子函数是什么 执行顺序是什么

路由钩子的执行流程, 钩子函数种类有:全局守卫、路由守卫、组件守卫

**完整的导航解析流程:**

1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

#### 20 vue-router 动态路由是什么 有什么问题

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 User 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。那么，我们可以在 vue-router 的路由路径中使用“动态路径参数”(dynamic segment) 来达到这个效果：

```javascript
const User = {
  template: "<div>User</div>",
};

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: "/user/:id", component: User },
  ],
});
复制代码
```

> 问题:vue-router 组件复用导致路由参数失效怎么办？

解决方法：

1.通过 watch 监听路由参数再发请求

```javascript
watch: { //通过watch来监听路由变化

 "$route": function(){
 this.getData(this.$route.params.xxx);
 }
}
复制代码
```

2.用 :key 来阻止“复用”

```html
<router-view :key="$route.fullPath" />
复制代码
```

#### 21 谈一下对 vuex 的个人理解

vuex 是专门为 vue 提供的全局状态管理系统，用于多个组件中数据共享、数据缓存等。（无法持久化、内部核心原理是通过创造一个全局实例 new Vue）

![vuex.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb545e2edc0a4dcb94a412db0625799c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) 主要包括以下几个模块：

- State：定义了应用状态的数据结构，可以在这里设置默认的初始状态。
- Getter：允许组件从 Store 中获取数据，mapGetters 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性。
- Mutation：是唯一更改 store 中状态的方法，且必须是同步函数。
- Action：用于提交 mutation，而不是直接变更状态，可以包含任意异步操作。
- Module：允许将单一的 Store 拆分为多个 store 且同时保存在单一的状态树中。

#### 22 Vuex 页面刷新数据丢失怎么解决

需要做 vuex 数据持久化 一般使用本地存储的方案来保存数据 可以自己设计存储方案 也可以使用第三方插件

推荐使用 vuex-persist 插件，它就是为 Vuex 持久化存储而生的一个插件。不需要你手动存取 storage ，而是直接将状态保存至 cookie 或者 localStorage 中

#### 23 Vuex 为什么要分模块并且加命名空间

**模块**:由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块。

**命名空间**：默认情况下，模块内部的 action、mutation 和 getter 是注册在全局命名空间的——这样使得多个模块能够对同一 mutation 或 action 作出响应。如果希望你的模块具有更高的封装度和复用性，你可以通过添加 namespaced: true 的方式使其成为带命名空间的模块。当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。

#### 24 使用过 Vue SSR 吗？说说 SSR

SSR 也就是服务端渲染，也就是将 Vue 在客户端把标签渲染成 HTML 的工作放在服务端完成，然后再把 html 直接返回给客户端。

**优点：**

SSR 有着更好的 SEO、并且首屏加载速度更快

**缺点：** 开发条件会受到限制，服务器端渲染只支持 beforeCreate 和 created 两个钩子，当我们需要一些外部扩展库时需要特殊处理，服务端渲染应用程序也需要处于 Node.js 的运行环境。

服务器会有更大的负载需求

#### 25 vue 中使用了哪些设计模式

1.工厂模式 - 传入参数即可创建实例

虚拟 DOM 根据参数的不同返回基础标签的 Vnode 和组件 Vnode

2.单例模式 - 整个程序有且仅有一个实例

vuex 和 vue-router 的插件注册方法 install 判断如果系统存在实例就直接返回掉

3.发布-订阅模式 (vue 事件机制)

4.观察者模式 (响应式数据原理)

5.装饰模式: (@装饰器的用法)

6.策略模式 策略模式指对象有某个行为,但是在不同的场景中,该行为有不同的实现方案-比如选项的合并策略

...其他模式欢迎补充

#### 26 你都做过哪些 Vue 的性能优化

> 这里只列举针对 Vue 的性能优化 整个项目的性能优化是一个大工程 可以另写一篇性能优化的文章 哈哈

- 对象层级不要过深，否则性能就会差
- 不需要响应式的数据不要放到 data 中（可以用 Object.freeze() 冻结数据）
- v-if 和 v-show 区分使用场景
- computed 和 watch 区分使用场景
- v-for 遍历必须加 key，key 最好是 id 值，且避免同时使用 v-if
- 大数据列表和表格性能优化-虚拟列表/虚拟表格
- 防止内部泄漏，组件销毁后把全局变量和事件销毁
- 图片懒加载
- 路由懒加载
- 第三方插件的按需引入
- 适当采用 keep-alive 缓存组件
- 防抖、节流运用
- 服务端渲染 SSR or 预渲染

------

### 困难

#### 27 Vue.mixin 的使用场景和原理

在日常的开发中，我们经常会遇到在不同的组件中经常会需要用到一些相同或者相似的代码，这些代码的功能相对独立，可以通过 Vue 的 mixin 功能抽离公共的业务逻辑，原理类似“对象的继承”，当组件初始化时会调用 mergeOptions 方法进行合并，采用策略模式针对不同的属性进行合并。当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。

相关代码如下

```javascript
export default function initMixin(Vue){
  Vue.mixin = function (mixin) {
    //   合并对象
      this.options=mergeOptions(this.options,mixin)
  };
}
};

// src/util/index.js
// 定义生命周期
export const LIFECYCLE_HOOKS = [
  "beforeCreate",
  "created",
  "beforeMount",
  "mounted",
  "beforeUpdate",
  "updated",
  "beforeDestroy",
  "destroyed",
];

// 合并策略
const strats = {};
// mixin核心方法
export function mergeOptions(parent, child) {
  const options = {};
  // 遍历父亲
  for (let k in parent) {
    mergeFiled(k);
  }
  // 父亲没有 儿子有
  for (let k in child) {
    if (!parent.hasOwnProperty(k)) {
      mergeFiled(k);
    }
  }

  //真正合并字段方法
  function mergeFiled(k) {
    if (strats[k]) {
      options[k] = strats[k](parent[k], child[k]);
    } else {
      // 默认策略
      options[k] = child[k] ? child[k] : parent[k];
    }
  }
  return options;
}
复制代码
```

Vue.mixin 原理详解 [传送门](https://juejin.cn/post/6951671158198501383)

#### 28 nextTick 使用场景和原理

nextTick 中的回调是在下次 DOM 更新循环结束之后执行的延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。主要思路就是采用微任务优先的方式调用异步方法去执行 nextTick 包装的方法

相关代码如下

```javascript
let callbacks = [];
let pending = false;
function flushCallbacks() {
  pending = false; //把标志还原为false
  // 依次执行回调
  for (let i = 0; i < callbacks.length; i++) {
    callbacks[i]();
  }
}
let timerFunc; //定义异步方法  采用优雅降级
if (typeof Promise !== "undefined") {
  // 如果支持promise
  const p = Promise.resolve();
  timerFunc = () => {
    p.then(flushCallbacks);
  };
} else if (typeof MutationObserver !== "undefined") {
  // MutationObserver 主要是监听dom变化 也是一个异步方法
  let counter = 1;
  const observer = new MutationObserver(flushCallbacks);
  const textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true,
  });
  timerFunc = () => {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
} else if (typeof setImmediate !== "undefined") {
  // 如果前面都不支持 判断setImmediate
  timerFunc = () => {
    setImmediate(flushCallbacks);
  };
} else {
  // 最后降级采用setTimeout
  timerFunc = () => {
    setTimeout(flushCallbacks, 0);
  };
}

export function nextTick(cb) {
  // 除了渲染watcher  还有用户自己手动调用的nextTick 一起被收集到数组
  callbacks.push(cb);
  if (!pending) {
    // 如果多次调用nextTick  只会执行一次异步 等异步队列清空之后再把标志变为false
    pending = true;
    timerFunc();
  }
}
复制代码
```

nextTick 原理详解 [传送门](https://juejin.cn/post/6939704519668432910#heading-4)

#### 29 keep-alive 使用场景和原理

keep-alive 是 Vue 内置的一个组件，可以实现组件缓存，当组件切换时不会对当前组件进行卸载。

- 常用的两个属性 include/exclude，允许组件有条件的进行缓存。
- 两个生命周期 activated/deactivated，用来得知当前组件是否处于活跃状态。
- keep-alive 的中还运用了 LRU(最近最少使用) 算法，选择最近最久未使用的组件予以淘汰。

相关代码如下

```javascript
export default {
  name: "keep-alive",
  abstract: true, //抽象组件

  props: {
    include: patternTypes, //要缓存的组件
    exclude: patternTypes, //要排除的组件
    max: [String, Number], //最大缓存数
  },

  created() {
    this.cache = Object.create(null); //缓存对象  {a:vNode,b:vNode}
    this.keys = []; //缓存组件的key集合 [a,b]
  },

  destroyed() {
    for (const key in this.cache) {
      pruneCacheEntry(this.cache, key, this.keys);
    }
  },

  mounted() {
    //动态监听include  exclude
    this.$watch("include", (val) => {
      pruneCache(this, (name) => matches(val, name));
    });
    this.$watch("exclude", (val) => {
      pruneCache(this, (name) => !matches(val, name));
    });
  },

  render() {
    const slot = this.$slots.default; //获取包裹的插槽默认值
    const vnode: VNode = getFirstComponentChild(slot); //获取第一个子组件
    const componentOptions: ?VNodeComponentOptions =
      vnode && vnode.componentOptions;
    if (componentOptions) {
      // check pattern
      const name: ?string = getComponentName(componentOptions);
      const { include, exclude } = this;
      // 不走缓存
      if (
        // not included  不包含
        (include && (!name || !matches(include, name))) ||
        // excluded  排除里面
        (exclude && name && matches(exclude, name))
      ) {
        //返回虚拟节点
        return vnode;
      }

      const { cache, keys } = this;
      const key: ?string =
        vnode.key == null
          ? // same constructor may get registered as different local components
            // so cid alone is not enough (#3269)
            componentOptions.Ctor.cid +
            (componentOptions.tag ? `::${componentOptions.tag}` : "")
          : vnode.key;
      if (cache[key]) {
        //通过key 找到缓存 获取实例
        vnode.componentInstance = cache[key].componentInstance;
        // make current key freshest
        remove(keys, key); //通过LRU算法把数组里面的key删掉
        keys.push(key); //把它放在数组末尾
      } else {
        cache[key] = vnode; //没找到就换存下来
        keys.push(key); //把它放在数组末尾
        // prune oldest entry  //如果超过最大值就把数组第0项删掉
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode);
        }
      }

      vnode.data.keepAlive = true; //标记虚拟节点已经被缓存
    }
    // 返回虚拟节点
    return vnode || (slot && slot[0]);
  },
};
复制代码
```

> 扩展补充：LRU 算法是什么？

![lrusuanfa.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa4a49bd77234467a53cd3b62c5bd135~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

LRU 的核心思想是如果数据最近被访问过，那么将来被访问的几率也更高，所以我们将命中缓存的组件 key 重新插入到 this.keys 的尾部，这样一来，this.keys 中越往头部的数据即将来被访问几率越低，所以当缓存数量达到最大值时，我们就删除将来被访问几率最低的数据，即 this.keys 中第一个缓存的组件。

#### 30 Vue.set 方法原理

了解 Vue 响应式原理的同学都知道在两种情况下修改数据 Vue 是不会触发视图更新的

1.在实例创建之后添加新的属性到实例上（给响应式对象新增属性）

2.直接更改数组下标来修改数组的值

Vue.set 或者说是$set 原理如下

因为响应式数据 我们给对象和数组本身都增加了__ob__属性，代表的是 Observer 实例。当给对象新增不存在的属性 首先会把新的属性进行响应式跟踪 然后会触发对象__ob__的 dep 收集到的 watcher 去更新，当修改数组索引时我们调用数组本身的 splice 方法去更新数组

相关代码如下

```typescript
export function set(target: Array | Object, key: any, val: any): any {
  // 如果是数组 调用我们重写的splice方法 (这样可以更新视图)
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);
    target.splice(key, 1, val);
    return val;
  }
  // 如果是对象本身的属性，则直接添加即可
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val;
  }
  const ob = (target: any).__ob__;

  // 如果不是响应式的也不需要将其定义成响应式属性
  if (!ob) {
    target[key] = val;
    return val;
  }
  // 将属性定义成响应式的
  defineReactive(ob.value, key, val);
  // 通知视图更新
  ob.dep.notify();
  return val;
}
复制代码
```

响应式数据原理详解 [传送门](https://juejin.cn/post/6935344605424517128)

#### 31 Vue.extend 作用和原理

官方解释：Vue.extend 使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象。

其实就是一个子类构造器 是 Vue 组件的核心 api 实现思路就是使用原型继承的方法返回了 Vue 的子类 并且利用 mergeOptions 把传入组件的 options 和父类的 options 进行了合并

相关代码如下

```javascript
export default function initExtend(Vue) {
  let cid = 0; //组件的唯一标识
  // 创建子类继承Vue父类 便于属性扩展
  Vue.extend = function (extendOptions) {
    // 创建子类的构造函数 并且调用初始化方法
    const Sub = function VueComponent(options) {
      this._init(options); //调用Vue初始化方法
    };
    Sub.cid = cid++;
    Sub.prototype = Object.create(this.prototype); // 子类原型指向父类
    Sub.prototype.constructor = Sub; //constructor指向自己
    Sub.options = mergeOptions(this.options, extendOptions); //合并自己的options和父类的options
    return Sub;
  };
}
复制代码
```

Vue 组件原理详解 [传送门](https://juejin.cn/post/6954173708344770591)

#### 32 写过自定义指令吗 原理是什么

指令本质上是装饰器，是 vue 对 HTML 元素的扩展，给 HTML 元素增加自定义功能。vue 编译 DOM 时，会找到指令对象，执行指令的相关方法。

自定义指令有五个生命周期（也叫钩子函数），分别是 bind、inserted、update、componentUpdated、unbind

```markdown
1. bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。

2. inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。

3. update：被绑定于元素所在的模板更新时调用，而无论绑定值是否变化。通过比较更新前后的绑定值，可以忽略不必要的模板更新。

4. componentUpdated：被绑定元素所在模板完成一次更新周期时调用。

5. unbind：只调用一次，指令与元素解绑时调用。
复制代码
```

**原理**

1.在生成 ast 语法树时，遇到指令会给当前元素添加 directives 属性

2.通过 genDirectives 生成指令代码

3.在 patch 前将指令的钩子提取到 cbs 中,在 patch 过程中调用对应的钩子

4.当执行指令对应钩子函数时，调用对应指令定义的方法

#### 33 Vue 修饰符有哪些

**事件修饰符**

- .stop 阻止事件继续传播
- .prevent 阻止标签默认行为
- .capture 使用事件捕获模式,即元素自身触发的事件先在此处处理，然后才交由内部元素进行处理
- .self 只当在 event.target 是当前元素自身时触发处理函数
- .once 事件将只会触发一次
- .passive 告诉浏览器你不想阻止事件的默认行为

**v-model 的修饰符**

- .lazy 通过这个修饰符，转变为在 change 事件再同步
- .number 自动将用户的输入值转化为数值类型
- .trim 自动过滤用户输入的首尾空格

**键盘事件的修饰符**

- .enter
- .tab
- .delete (捕获“删除”和“退格”键)
- .esc
- .space
- .up
- .down
- .left
- .right

**系统修饰键**

- .ctrl
- .alt
- .shift
- .meta

**鼠标按钮修饰符**

- .left
- .right
- .middle

#### 34 Vue 模板编译原理

Vue 的编译过程就是将 template 转化为 render 函数的过程 分为以下三步

```
第一步是将 模板字符串 转换成 element ASTs（解析器）
第二步是对 AST 进行静态节点标记，主要用来做虚拟DOM的渲染优化（优化器）
第三步是 使用 element ASTs 生成 render 函数代码字符串（代码生成器）
复制代码
```

相关代码如下

```javascript
export function compileToFunctions(template) {
  // 我们需要把html字符串变成render函数
  // 1.把html代码转成ast语法树  ast用来描述代码本身形成树结构 不仅可以描述html 也能描述css以及js语法
  // 很多库都运用到了ast 比如 webpack babel eslint等等
  let ast = parse(template);
  // 2.优化静态节点
  // 这个有兴趣的可以去看源码  不影响核心功能就不实现了
  //   if (options.optimize !== false) {
  //     optimize(ast, options);
  //   }

  // 3.通过ast 重新生成代码
  // 我们最后生成的代码需要和render函数一样
  // 类似_c('div',{id:"app"},_c('div',undefined,_v("hello"+_s(name)),_c('span',undefined,_v("world"))))
  // _c代表创建元素 _v代表创建文本 _s代表文Json.stringify--把对象解析成文本
  let code = generate(ast);
  //   使用with语法改变作用域为this  之后调用render函数可以使用call改变this 方便code里面的变量取值
  let renderFn = new Function(`with(this){return ${code}}`);
  return renderFn;
}
复制代码
```

模板编译原理详解 [传送门](https://juejin.cn/post/6936024530016010276)

#### 35 生命周期钩子是如何实现的

Vue 的生命周期钩子核心实现是利用发布订阅模式先把用户传入的的生命周期钩子订阅好（内部采用数组的方式存储）然后在创建组件实例的过程中会一次执行对应的钩子方法（发布）

相关代码如下

```javascript
export function callHook(vm, hook) {
  // 依次执行生命周期对应的方法
  const handlers = vm.$options[hook];
  if (handlers) {
    for (let i = 0; i < handlers.length; i++) {
      handlers[i].call(vm); //生命周期里面的this指向当前实例
    }
  }
}

// 调用的时候
Vue.prototype._init = function (options) {
  const vm = this;
  vm.$options = mergeOptions(vm.constructor.options, options);
  callHook(vm, "beforeCreate"); //初始化数据之前
  // 初始化状态
  initState(vm);
  callHook(vm, "created"); //初始化数据之后
  if (vm.$options.el) {
    vm.$mount(vm.$options.el);
  }
};
复制代码
```

生命周期实现详解 [传送门](https://juejin.cn/post/6951671158198501383#heading-4)

#### 36 函数式组件使用场景和原理

函数式组件与普通组件的区别

```kotlin
1.函数式组件需要在声明组件是指定 functional:true
2.不需要实例化，所以没有this,this通过render函数的第二个参数context来代替
3.没有生命周期钩子函数，不能使用计算属性，watch
4.不能通过$emit 对外暴露事件，调用事件只能通过context.listeners.click的方式调用外部传入的事件
5.因为函数式组件是没有实例化的，所以在外部通过ref去引用组件时，实际引用的是HTMLElement
6.函数式组件的props可以不用显示声明，所以没有在props里面声明的属性都会被自动隐式解析为prop,而普通组件所有未声明的属性都解析到$attrs里面，并自动挂载到组件根元素上面(可以通过inheritAttrs属性禁止)
复制代码
```

优点 1.由于函数式组件不需要实例化，无状态，没有生命周期，所以渲染性能要好于普通组件 2.函数式组件结构比较简单，代码结构更清晰

使用场景：

一个简单的展示组件，作为容器组件使用 比如 router-view 就是一个函数式组件

“高阶组件”——用于接收一个组件作为参数，返回一个被包装过的组件

相关代码如下

```javascript
if (isTrue(Ctor.options.functional)) {
  // 带有functional的属性的就是函数式组件
  return createFunctionalComponent(Ctor, propsData, data, context, children);
}
const listeners = data.on;
data.on = data.nativeOn;
installComponentHooks(data); // 安装组件相关钩子 （函数式组件没有调用此方法，从而性能高于普通组件）
复制代码
```

#### 37 能说下 vue-router 中常用的路由模式实现原理吗

**hash 模式**

1. location.hash 的值实际就是 URL 中#后面的东西 它的特点在于：hash 虽然出现 URL 中，但不会被包含在 HTTP 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。
2. 可以为 hash 的改变添加监听事件

```javascript
window.addEventListener("hashchange", funcRef, false);
复制代码
```

每一次改变 hash（window.location.hash），都会在浏览器的访问历史中增加一个记录利用 hash 的以上特点，就可以来实现前端路由“更新视图但不重新请求页面”的功能了

> 特点：兼容性好但是不美观

**history 模式**

利用了 HTML5 History Interface 中新增的 pushState() 和 replaceState() 方法。

这两个方法应用于浏览器的历史记录站，在当前已有的 back、forward、go 的基础之上，它们提供了对历史记录进行修改的功能。这两个方法有个共同的特点：当调用他们修改浏览器历史记录栈后，虽然当前 URL 改变了，但浏览器不会刷新页面，这就为单页应用前端路由“更新视图但不重新请求页面”提供了基础。

> 特点：虽然美观，但是刷新会出现 404 需要后端进行配置

#### 38 diff 算法了解吗

![diff算法.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e3c68d1b0884d9ca0f8ffc5ee64a28e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

建议直接看 diff 算法详解 [传送门](https://juejin.cn/post/6953433215218483236)



作者：前端鲨鱼哥
链接：https://juejin.cn/post/6961222829979697165
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 从mini源码了解什么vue

Vue.js是一套构建用户界面的渐进式框架。 他最大的优势，也是单页面最大的优势，数据驱动与组件化。

首先我们mini的源码了解vue如何完成数据驱动。如图：

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/6/1732486e91c2ebf3~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

从图我们就可以简单的分析出什么叫MVVM。

MVVM, 实际上为M + V + VM。vue的框架就是一个内置的VM状态，而M就是我们的MODLE, V即是我们的视图。而通过我们的M，就能实现对V的控制，就是我们所说的数据驱动（模型控制视图）。ViewModel 的内容会实时展现在 View 层，前端开发者再也不必低效又麻烦，还耗性能（因为没有diff算法）地通过操纵 DOM 去更新视图。这就是一个从根源上，MVVM框架比传统MVC框架的优势。

我们进一步手写Mini版来了解vue，从源码了解什么是数据劫持。

首先构造一个vue实例。写过vue初始化的都知道，初始化时需要传入data，以及绑定元素标记el。我们把它储存起来。

```kotlin
class wzVue {
	constructor(options){
		this.$options = options;
		this.$data = options.data;
		this.$el = options.el;
	}
}
复制代码
```

首先看一下Observer的实现，以vue2.0为例，我们都知道数据劫持是通过Object.defineProperty。它自带监听get,set方法，我们可以用他实现一个简单的绑定。

```javascript
obsever(this.$data);

function obsever(){
      Object.defineProperty( obj, key, {
		get(){
			
		},
		set( newValue ){
			value = newValue;
		}
	})
}
复制代码
```

这里很简单，如果还是不明白怎么双向绑定，举个简单的栗子：

```javascript
<input type="text" v-modle="key" id="key"/>

// script
var data = {
    key: 5
    key2: 8
}
obsever(data);
data.key=6;//

function obsever(obj){
      Object.defineProperty( obj, key, {
		get(){
			
		},
		set( newValue ){
		    document.getElementById('key').val(newValue);//写死'key'先，下文会讲解
		}
	})
}

//写死'key'先，下文会讲解
document.getElementById('key').addEventListener( 'click', false, function(e){
    obj.key = e.target.value;
})
复制代码
```

这样实现了双向绑定，如果对象obj.key赋值，就会触发set方法，同步input的数据;如果页面手动输入值，则通过监听触发set，同步到对象obj的值。此时你可能有一个疑问，我们在vue赋值的时候，是直接修改上下文data数据的，并不是修改对对象的值, 也就是this.key=6。是的，vue源码中，先对data对象的数据进行了一次本地的数据劫持。如下文的proxyData。这样的:

this.key  ----> data.key(触发) --->实现数据劫持

```javascript
observer( data ){//监听data数据，双向绑定
    if( !data || typeof(data) !== 'object'){
		return;
	}
	Object.keys( data ).forEach( key => {
		this.observerData(key, data, data[key]);//监听data对象
		this.proxyData( key );
	})
}

observerData( key, obj, value ) {
	this.observer(key);
	const dep = new Dep();
	Object.defineProperty( obj, key, {
		get(){
		},
		set( newValue ){ //通知变化
		}
	})
}

proxyData(key){
	Object.defineProperty( this, key, {
		get(){
			return this.$data[key];
		},
		set( newValue ){	
			this.$data[key] = newValue;
		}			
	})
}
复制代码
```

两点需要强调的地方：

1)遍历data的属性，vue的源码是用了Object.keys。它能按顺序遍历出不同的属性，但是不同的浏览器中可能执行顺序不一样。

2)因为Object.defineProperty只能监听一层结构，所以，对于多层级的Object结构来讲，需要遍历去一层一层往下监听。

那如果连续赋值的，例如this.key = 1; this.key2 = 2; 上边的双向绑定代码是写死了“key"。

这时候是否发生了两次赋值？那么我们怎么知道，它触发的对象是哪个呢?这时候，vue的设计是设计了dep的概念，来存放每个监听对象的值。

```javascript
class Dep{
	constructor(){
		this.deps = [];
	}
	
	addDep(dep){
		this.deps.push(dep);
	}
	
	notiyDep(){
		this.deps.forEach(dep => {
			dep.update();
		})
	}
}
复制代码
```

这里不难理解。addDep既是为了有数据变化时，插入的“对象”，表示需要劫持。 notiyDep即是该对象，已经需要被更新，执行对应的update方法。

那么插入的对象是什么呢（数组的单体）？单体肯定，需要包含一个“dom”对象，还有对应监听的“data”对象，两者关系绑定，才能实现数据同步。这个“单体”，我们称呼它为“watcher”。

```kotlin
class Watcher{

	constructor( vm, key, initVal, cb ){
		this.vm = vm;//保存vue对象实例
		this.key = key;//保存绑定的key
		this.cb = cb;//同步两者的回调函数
		this.initVal = initVal;//初始化值
		this.vm[this.key];//触发对象的get方法
	}
	
	update(){
		this.cb.call( this.vm, this.vm[this.key], this.initVal );
	}

}
复制代码
```

截至目前为止，obsever还是没有跟Watcher关联上。在讲他们怎么关联上之前，我们再看看vue的设计思维，它是由Watcher添加订阅者，再由Dep添加变化。那么Watcher是怎么来的？从图中的关系，我们可以看出由页面解析出来的。这就是我们要讲的 Compile。 ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/7/17327453d3626a7d~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

Compile，首先有一个“初始化视图”的动作。

```kotlin
class Compile{
	
	constructor( el, vm ){
		this.$el  =  document.querySelector(el);
		this.$vm = vm;
		if (this.$el) {
			this.$fragment = this.getNodeChirdren( this.$el );
			this.$el.appendChild(this.$fragment);
		}
	}
	
	getNodeChirdren( el ){
		const frag = document.createDocumentFragment();
		let child;
		while( (child = el.firstChild )){
			frag.appendChild( child );
		}
		return frag;
	}
}
复制代码
```

这里应该不难理解，拿到template对象的id，遍历完之后，赋值显示在我们的el元素中。接下来我们重点讲Compile产生的Watcher。我们在Compile的原型中添加this.compile( this.$fragment);方法。对刚才拿到template的模版进行继续，看他用到哪些属性。

```ini
compile( el ){
	const childNodes = el.childNodes;
	Array.from(childNodes).forEach( node => {
		if( node.nodeType == 1 ) {//1为元素节点
			const nodeAttrs = node.attributes;
			Array.from(nodeAttrs).forEach( attr => {
				const attrName = attr.name;//属性名称
				const attrVal = attr.value;//属性值
				if( attrName == "v-modle"){
				   	this.zDir_model( node, attrVal );
				}
			})
		} else if( node.nodeType == 2 ){//2为属性节点
			console.log("nodeType=====22");
		} else if( node.nodeType == 3 ){//3为文本节点
			this.compileText( node );
		}
		// 递归子节点
		if (node.childNodes && node.childNodes.length > 0) {
			this.compile(node);
		}
	})
}
复制代码
```

如果你对childNodes，nodeType，nodeList还是一脸懵逼，建议移步到： 关于DOM和BOM知识点汇总： [juejin.cn/post/684668…](https://juejin.cn/post/6846687586430222343)

从上边的mini源码可以看出，compile遍历el的所有子元素，如果是文本类型，我们就进行文本解析compileText。如果是input需要双向绑定，我们就进行zDir_model解析。

```ini
compileText( node ){
	if( typeof( node.textContent ) !== 'string' ) {
		return "";
	}
	const reg = /({{(.*)}})/;
	const reg2 = /[^/{/}]+/;
	const key = String((node.textContent).match(reg)).match(reg2);//获取监听的key
	const initVal = node.textContent;//记录原文本第一次的数据
    updateText( node, this.$vm[key], initVal );
}

updateText( node, value, initVal ){
	var reg = /{{(.*)}}/ig;
	var replaceStr = String( initVal.match(reg) );
	var result = initVal.replace(replaceStr, value );
	node.textContent = result;
	new Watcher( this.$vm, key, initVal, function( value, initVal ){
		updateText( node, value, initVal  );
	});
}
复制代码
```

我们再看看compileText的源码，大概意思为，获取到文本例如“我的名字{{name}}”的key，即为name。然后name进行初始化赋值updateText， updateText的初始化结束后，添加订阅数据变化，绑定更新函数Watcher。

而Watcher，正是绑定dep跟compile的桥梁。我们修改一下添加到dep跟Watcher的代码：

```kotlin
observerData( key, obj, value ) {
	this.observer(key);
	const dep = new Dep();
	Object.defineProperty( obj, key, {
		get(){
		    Dep.target && dep.addDep(Dep.target);//添加的代码+++++++++++++++++
			return value;
		},
		set( newValue ){ //通知变化
		    if (newValue === value) {
			  return;
			}
			value = newValue;
			//通知变化
			dep.notiyDep();//添加的代码+++++++++++++++++
		}
	})
}


class Watcher{

	constructor( vm, key, initVal, cb ){
		this.vm = vm;
		this.key = key;
		this.cb = cb;
		this.initVal = initVal;
		Dep.target = this;//添加的代码+++++++++++++++++
		this.vm[this.key];
		Dep.target = null;
	}
	
	update(){
		this.cb.call( this.vm, this.vm[this.key], this.initVal );
	}

}
复制代码
```

这样的话，我们在新增一个Watcher的过程中，将此时的整个Watcher的this对象赋值到Dep.target中。这时候我们再调用一下this.vm[this.key]。vm即是vue实例对象，所以，Watcher的this.vm[this.key]，即是vue实例中的，this.key。而我们的key已经通过Object.defineProperty监听，此时就会进入到Object.defineProperty的get方法中， Dep.target 此时不为空，所以dep.addDep(Dep.target)，即是watcher添加订阅者到dep中。

这时候如果数据发生变化，即调用set方法，然后dep.notiyDep，notiyDep就会通知，由文本解析的例如{{key}}的watcher重新更新一遍值，即完成了双向绑定。

如果是v-modle的话，即在解析时，每个对象多加一个监听，然后主动调用set方法。

```ini
node.addEventListener("input", e => {
	  vm[value] = e.target.value;
	});
	
复制代码
```

这就是vue整个双向绑定的大致流程，所谓的数据驱动。

然后他有一个很大的**缺陷**，这个缺陷是，他知道驱动对象，却无法对数组进行驱动 **（实际上也行）** 。这里vue的作者用了另外一种思维去解决这个问题。他重写了数组的原型，把数组的'push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'的方法重写了一遍。也就是当你数组使用了这7个方法时，vue重写的方法，会帮你变化中放入dep中。

这 个 **（实际上也行）** 其实也很有学问。上述说，vue2.0无法对数组进行数据监听，其实真实的的测试中，Object.defineProperty是可以监听到数组变化的。但是只能在已有的长度中，不能对其加长的长度。那你这时候可能会有疑问，那我们重写array的push方法就够了，为什么要重写7个呢？好吧，我也曾经有这样的疑惑。后续，曾在帖子上，看到过vue的笔者回复过，印象中是这么说的：Object.defineProperty对数组的监听，消耗性能大于效果。也就是说，本来Object.defineProperty，为了提升效率而产生，现在用在数组上，反而降低了效率，那不如干脆拒绝使用他。

于是，又有了vue-cli3.0数据劫持的改造。

那么vue3.0是怎么实现数据劫持的呢？

3.0中双向绑定已经不再是使用Object.defineProperty。而是proxy。proxy的引入，更高的效率，一方面解决了数组方面的问题，我们可以简单看一下mini源码的改造：

```kotlin
proxyData( data ){//监听data数据，双向绑定
	if( !data || typeof(data) !== 'object'){
		return;
	}
	const _this = this;
	const handler  = {
		set( target, key, value ) {
			const rest = Reflect.set(target, key, value);
			_this.$binding[key].map( item =>  {
				item.update();
			})
			return rest;
		}
	}
	this.$data = new Proxy( data, handler );
}
复制代码
```

vue的mini源码解析到此为止，如还有不明白的地方可留言。 可需要源码可进入github查看：[github.com/zhuangweizh…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fzhuangweizhan%2FcodeShare)

### vue的特性是什么

#### 双向绑定

由上述mini源码，我们可以知道vue的数据驱动。MVVM相比MVC模式, 没有频繁的操作dom值，在开发中无疑时更高效的灵活页面的触发。可让我们专注与逻辑js的抒写，而具体的页面变化，交给VM区处理。

#### diff算法

我们都知道，js执行的效率高于dom渲染的效率。如果我们能提前通过js算出不一致的地方，再最后去“渲染”最终的差异。明显的增加效益。

我们列出diff算法的三步曲：

- 1）通过虚拟dom渲染对象
- 2）对比两个虚拟的差异
- 3）根据差异进行渲染

#### 全局混入mixins

mixins 选项接收一个混入对象的数组。而vue正是利用他来扩展vue的实例。

我们的全局方法等，都可以利用mixins快速的套入vue实例。

#### 完善的生命周期

十一个生命周期，create, mount, update, activated, destroyed。分别前后。最后还有v2.5.0版本的errorCaptured。 完善的生命周期的更适合，程序顺序的正确执行。

#### 丰富的组件传递

props, emit, slot，provide/inject，attrs/listeners，EventBus emit/on，parent / children与 ref

### vue的优势是什么

也是你为什么选择vue的原因

#### 易学上手

笔者曾是一名jq的前端小杂，入门这些玩意，个人觉得他们的难度级别（仅限于api）：

jq < 原生小程序 <  vue系列  <  angurle系列 < react系列

vue是刚开始一边看着api就可以撸出来的项目。

#### 活跃的社区

也许每个框架都有自身的bug。有bug不可怕，怕的是没有解决方案。而vue中，你卡到问题点，但自己没有能力解决时，活跃的社区会给你答案。

#### 完善的第三方插件

支持axios, webpack， sass，elemnt-ui，vuex, router 等第三方插件。

#### 支持客户端全家桶

vue有着脚手架，ssr的nuxt框架，app版本的weex， 小程序多端开发uniapp。

可谓学好vue，吃遍前端全家桶。

------

**最后：也许你觉得，上述react都支持。好的吧，的确是，晚些汇总完react的文章，再写一篇对比。**

### vue-cli包含了什么

vue-cli脚手架，帮我们做了什么。vue-cli3.0开始，已经成为可选择性的插件。我们分析一下各个插件的作用。

#### webpack

[blog.guowenfh.com/2016/03/24/…](https://link.juejin.cn?target=https%3A%2F%2Fblog.guowenfh.com%2F2016%2F03%2F24%2Fvue-webpack-03-config%2F)

webpack，打包所有的“脚本”。脚手架已经帮我们通过webpack做了很多默认的loader。

我们项目中，不同的文件，经过编译输出最终的html,js,css，都是经过webpack。

例如，编译 ES2015 或 TypeScript 模块成 ES5 CommonJS 的模块；

再例如：编译 SASS 文件成 CSS，然后把生成的CSS插入到 Style 标签内，然后再转译成 JavaScript 代码段，处理在 HTML 或 CSS 文件中引用的图片文件，根据配置路径把它们移动到任意位置，根据 MD5 hash 命名。

因此，我们可以不同文件，找在webpack不同的编译器，如vue有vue-loader,脚手架帮我们引入了。如sass有sass-loader，基本npm或者yarn生态圈中，已经有前端你所有见过的loader。也许还有没有？没关系，我们可以自己写一个。

来个简单的需求：开发环境过滤掉所有的打印。

这要是在传统的项目，没有经过编译器，这是有多大的工作量。当有了我们的webpack或者gulp等，他仅仅只是几句代码的问题。我们来看一下webpack的实现：

配置文件：

```ini
const fs = require('fs');

function wzJsLoader(source) {
    /*过滤输出*/
    const mode = process.env.NODE_ENV === 'production'
    if( mode ){//正式环境
        source = source.replace(/console.log\(.*?\);/ig, value => "" );
    }
    return source;
};

module.exports = wzJsLoader;
复制代码
```

这样，我们就轻松定了一个自己的loader。在wepback.config.js，加上我们对应的loader，轻松解决问题

```javascript
{
    test: /\.js$/, //js文件加载器
    exclude: /node_modules/,
    use: [
        {
          loader: 'babel-loader?cacheDirectory=ture',
          options: {
            presets: ['@babel/preset-env']
          }
        },
        {
            loader: require.resolve("./myWebPackConfig/wz-js-loader"),//添加的
            options: { name: __dirname  },
        },
    ]
  },
  
复制代码
```

webpack是个很有难度的东西，本文就不继续简介，简单了解webpack的配置，以及如何写好Loader跟plugins等。如果你还有精力深入，webpack的执行机制，如何打包成文件，他的生命周期等，都可以深入挑战，如果研究透彻，相信你的实力不一般。

#### axios

axios，网络请求工具。提到网络请求工具，你肯定了解从$.ajax，fetch、axios。下边此次讲一下**他们的发展史以及优缺点（具体什么时候，会在下文的“vue项目的二次封装”中讲解）**

$.ajax，相信早期进入前端领域的人，都大为喜欢。他基于jquery，对原生XHR的封装，还支持JSONP，非常方便。 他的有点包括，无需要通过刷新页面更新数据，支持异步与服务器通信，而且规范被广泛支持。

当年可谓如“诺基亚”一般存在。可惜“诺基亚”后来跌下神坛，$.ajax在网络请求中也遭受的同样的待遇。

那么淘汰$.ajax的根本原因是什么呢？

因为引入的单页面框架，如vue的mvvn架构，或者是只有m的react，他们都属于js驱动Html。这涉及到控制dom刷新的过程。es5可以利用callback, 或者generater的迭代器模式进行处理。但是还不理想。所以es6引入了promise的概念。

所以，以返回promise的单位的异步控制进程逐步发展。

一方面，.ajax没有改进，他依然我行我素的不支持promise。这对“新”前端的理念很不符，我们无法用.ajax没有改进，他依然我行我素的不支持promise。这对“新”前端的理念很不符，我们无法用.ajax没有改进，他依然我行我素的不支持promise。这对“新”前端的理念很不符，我们无法用.ajax来完成异步操作（除非回调地狱，写过大项目的都知道定位问题太难了）。

另一方面，他还需要引入jquery来实现。我们都知道新框架，都基本脱离了jq。

SO,fetch就这样产生了。解决了ajax无法返回promise的问题。开始让人抛弃$.ajax。

fetch号称是$.ajax的替代品，它的API是基于Promise设计的，旧版本的浏览器不支持Promise，需要使用polyfill es6-promise

然而，fetch貌似是为解决返回Promise而产生的，并没有注意其他网络请求工具该做的细节，他虽然支持promise, 但暴露了太多的问题：

1）fetch只对网络请求报错，对400，500都当做成功的请求，服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject。

2）fetch默认不会带cookie，需要添加配置项： fetch(url, {credentials: 'include'})

3）fetch不支持abort，不支持超时控制，使用setTimeout及Promise.reject的实现的超时控制并不能阻止请求过程继续在后台运行，造成了流量的浪费

4）fetch没有办法原生监测请求的进度，而XHR可以

因此，axios正式入场。他重新基于xhr封装，支持返回Promise, 也解决了fetch的弊端。

反问：知道jquery,fetch,axios的区别了吗？

#### vue-router

在没有“路由”的概念时，我们通常讲“页面路径”。如果你经历过spring mvc通过action映射到html页面的时代，那么恭喜 ，你已经使用过路由。他属于后台路由。后台的路由，可以简单的理解成一个路径的映射。

那么有后台路由，就会有前端路由。没错，带来质的改变，就是前端路由。那么他带来的优势是什么。

前端路由，又分hash模式跟history模式。我们用两张图来简单的说明一下，前端路由的原理：

##### hash模式

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/8/1732edc5d30d6887~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

hash（#）是URL 的锚点，代表的是网页中的一个位置，单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页，也就是说hash 出现在 URL 中，但不会被包含在 http 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面；同时每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用”后退”按钮，就可以回到上一个位置；所以说Hash模式通过锚点值的改变，根据不同的值，渲染指定DOM位置的不同数据。hash 模式的原理是 onhashchange 事件(监测hash值变化)，可以在 window 对象上监听这个事件。

优势呢？是不是很明显？如果没有使用异步加载，我们的已经可以不需要经过后台，直接仅是页面的“锚点”切换。

##### history模式

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/8/1732edc88a0935a7~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

history模式充分利用了html5 history interface 中新增的 pushState() 和 replaceState() 方法。这两个方法应用于浏览器记录栈，在当前已有的 back、forward、go 基础之上，它们提供了对历史记录修改的功能。只是当它们执行修改时，虽然改变了当前的 URL ，但浏览器不会立即向后端发送请求。

history模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

**此外，**vue的路由，还支持嵌套（多级）路由，支持路由动态配置，命名视图（同一页面多个路由），路由守卫, 过渡动态效果等，可谓功能十分之强大，考虑比较齐全，在此每个列举一个简单的栗子：

路由动态配置：

```arduino
const router = new VueRouter({
  routes: [
   动态路径参数 以冒号开头
    { path: '/detail/:id', component: Detail }
  ]
})
复制代码
```

嵌套（多级）路由： const router = new VueRouter({

```yaml
routes: [
        { path: '/detail/', component: User,
              children: [
                {
                  path: 'product',
                  component: Product  //二级嵌套路由
                },
              ]
        }
   ]
})
复制代码
```

命名视图：

```sql
<router-view></router-view>
<router-view name="a"></router-view>
<router-view name="b"></router-view>

const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: componentsDefulat,
        a: componentsA,
        b: componentsB
      }
    }
  ]
})
复制代码
```

路由守卫:

```javascript
router.beforeEach((to, from, next) => {
  // ...
})
复制代码
```

动态效果：

```xml
<transition>
  <router-view></router-view>
</transition>
复制代码
```

#### sass/less

sass跟less，两者都是CSS预处理器的佼佼者。

为什么要使用CSS预处理器？

CSS有具体以下几个**缺点**：

1.语法不够强大，比如无法嵌套书写，导致模块化开发中需要书写很多重复的选择器；

2.没有变量和合理的样式复用机制，使得逻辑上相关的属性值必须以字面量的形式重复输出，导致难以维护。

Less和Sass在语法上有些共性，比如下面这些：

- 1、混入(Mixins)——class中的class；
- 2、参数混入——可以传递参数的class，就像函数一样；
- 3、嵌套规则——Class中嵌套class，从而减少重复的代码；
- 4、运算——CSS中用上数学；
- 5、颜色功能——可以编辑颜色；
- 6、名字空间(namespace)——分组样式，从而可以被调用；
- 7、作用域——局部修改样式；
- 8、JavaScript 赋值——在CSS中使用JavaScript表达式赋值。

再说一下两者的区别：

- 1.Less环境较Sass简单，使用起来较Sass简单

- 2.从功能出发，Sass较Less略强大一些 (1) sass有变量和作用域。

  (2) sass有函数的概念；

  (3) sass可以进行进程控制。例如： -条件：@if @else； -循环遍历：@for @each @while

  (4) sass又数据结构类型： -list类型=数组； -map类型=object； 其余的也有string、number、function等类型

- 3.Less与Sass处理机制不一样

- 前者是通过客户端处理的，后者是通过服务端处理，相比较之下前者解析会比后者慢一点。而且sass会产生服务器压力。

#### vuex

vuex官方概念：Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。

看到这里你可能会有疑问，我们传统的框架上，localstore, session , cookies,不就以及解决问题了么。

没错。他们是解决了本地存储的问题。但是vue是单页面架构，需要数据驱动。 session , cookies无法触发数据驱动。这时候不得引入一个可以监听的容易。小型项目可能直接用store，或者页面与页面直接可以用props传递

我们在使用Vue.js开发复杂的应用时，经常会遇到多个组件共享同一个状态，亦或是多个组件会去更新同一个状态，在应用代码量较少的时候，我们可以组件间通信去维护修改数据，或者是通过事件总线来进行数据的传递以及修改。但是当应用逐渐庞大以后，代码就会变得难以维护，从父组件开始通过prop传递多层嵌套的数据由于层级过深而显得异常脆弱，而事件总线也会因为组件的增多、代码量的增大而显得交互错综复杂，难以捋清其中的传递关系。

那么为什么我们不能将数据层与组件层抽离开来呢？把数据层放到全局形成一个单一的Store，组件层变得更薄，专门用来进行数据的展示及操作。所有数据的变更都需要经过全局的Store来进行，形成一个单向数据流，使数据变化变得“可预测”。

简单说一下他的工作流程：

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/9/173326eca54c4b44~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

图文相信已经非常清晰vuex的工作流程。简单的简述一下api：

**state** 简单的理解就是vuex数据的储存对象。

**getters** getter 会暴露为 state 对象，你可以以属性的形式访问这些值：

**actions** Action 类似于 mutation，不同在于： Action 提交的是 mutation，而不是直接变更状态。 Action 可以包含任意异步操作。

**mutations** 每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。 mutations可以直接改变state的状态。 mutations 不可以包含任意异步操作

**module** Vuex 太大时，允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter。

vuex的使用，很简单。但是灵活时候，可能还需要进一步的了解源码。vuex的原理其实跟vue有点像。

如需要看vuex源码，可通过：[github.com/zhuangweizh…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fzhuangweizhan%2FcodeShare)

#### element ui/vux

即UI库的选择

vue的火热，离不开vue社区的火热。就常规的项目，如果公司不是要求特别高，基本各种UI库，已经不需要你写样式（前端最烦的就是写样式没意见吧）。

这里就不做UI库如何搭建的文章，有兴趣可以关注，后续我会写一篇专门搭建UI库的。

这里介绍一下vue火热的UI库吧。

其中，移动端笔者推荐vant，管理后台推荐element。

- **elementUI**： [element.eleme.io/#/zh-CN](https://link.juejin.cn?target=https%3A%2F%2Felement.eleme.io%2F%23%2Fzh-CN)
- **vant**： [youzan.github.io/vant/#/zh-C…](https://link.juejin.cn?target=https%3A%2F%2Fyouzan.github.io%2Fvant%2F%23%2Fzh-CN%2F)
- **iviewui**： [www.iviewui.com/](https://link.juejin.cn?target=https%3A%2F%2Fwww.iviewui.com%2F)
- **BootstrapVue**： [bootstrap-vue.org/docs](https://link.juejin.cn?target=https%3A%2F%2Fbootstrap-vue.org%2Fdocs)
- **Vux**： [vux.li/#/](https://link.juejin.cn?target=https%3A%2F%2Fvux.li%2F%23%2F)
- **Mint UI**： [mint-ui.github.io/#!/zh-cn](https://link.juejin.cn?target=http%3A%2F%2Fmint-ui.github.io%2F%23!%2Fzh-cn)

### vue项目的二次封装

##### axios的封装

上文讲解过axios的由来以及优缺点，这里谈谈axios在vue项目的使用。

1)请求拦截

比如我们的请求接口，全局都需要做token验证。我们可以在请求钱做好token雁阵。如果存在，则请求头自动添加token。

```ini
axios.interceptors.request.use(    
    config => {        
        // 每次发送请求之前判断vuex中是否存在token        
        // 如果存在，则统一在http请求的header都加上token，这样后台根据token判断你的登录情况
        // 即使本地存在token，也有可能token是过期的，所以在响应拦截器中要对返回状态进行判断 
        const token = store.state.token;        
        token && (config.headers.token = token);        
        return config;    
    },    
    error => {        
        return Promise.error(error);    
})
复制代码
```

2)返回拦截

当程序异常的时候呢，接口有时候在特定的场景，或者是服务器异常的情况下，是否就让用户白白等待？ 如果有超时，错误返回机制，及时告知用户的，是不是用户好一点？这就是返回的拦截。

```javascript
axios.interceptors.response.use(    
    response => {   
        // 如果返回的状态码为200，说明接口请求成功，可以正常拿到数据     
        // 否则的话抛出错误
        if (response.status === 200) {            
            return Promise.resolve(response);        
        } else {            
            return Promise.reject(response);        
        }    
    },    
    // 服务器状态码不是2开头的的情况
    // 这里可以跟你们的后台开发人员协商好统一的错误状态码    
    // 然后根据返回的状态码进行一些操作，例如登录过期提示，错误提示等等
    // 下面列举几个常见的操作，其他需求可自行扩展
    error => {    
            alert("数据异常，请稍后再试或联系管理员");
            return Promise.reject(error.response);
        }
    }    
});
复制代码
```

3)以get为栗子

```javascript
export function get(url, params){    
    return new Promise((resolve, reject) =>{        
        axios.get(url, {            
            params: params        
        }).then(res => {
            resolve(res.data);
        }).catch(err =>{
            reject(err.data)        
    })    
});
复制代码
```

此外，对axios的使用还有想法的，建议查看一下axios全攻略： [ykloveyxk.github.io/2017/02/25/…](https://link.juejin.cn?target=https%3A%2F%2Fykloveyxk.github.io%2F2017%2F02%2F25%2Faxios%E5%85%A8%E6%94%BB%E7%95%A5%2F%23more)

#### 编译器改进

上文曾提到，vue-cli自带webpack。那么我们如何通过他，来改进我们的项目呢。

从环境区分，自带的引入，已经帮我们区分了环境，然后帮我们导入不同的loader跟Pulger等，基本已经是一个非常完善的编译器。

我们见到看一下dev的源码(添加了注释)，dev环境，实际上会运行dev-server.js文件该文件以express作为后端框架

```javascript
// nodejs环境配置
var config = require('../config')
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var opn = require('opn') //强制打开浏览器
var path = require('path')
var express = require('express')
var webpack = require('webpack')
var proxyMiddleware = require('http-proxy-middleware') //使用代理的中间件
var webpackConfig = require('./webpack.dev.conf') //webpack的配置

var port = process.env.PORT || config.dev.port //端口号
var autoOpenBrowser = !!config.dev.autoOpenBrowser //是否自动打开浏览器
var proxyTable = config.dev.proxyTable //http的代理url

var app = express() //启动express
var compiler = webpack(webpackConfig) //webpack编译

//webpack-dev-middleware的作用
//1.将编译后的生成的静态文件放在内存中,所以在npm run dev后磁盘上不会生成文件
//2.当文件改变时,会自动编译。
//3.当在编译过程中请求某个资源时，webpack-dev-server不会让这个请求失败，而是会一直阻塞它，直到webpack编译完毕
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})

//webpack-hot-middleware的作用就是实现浏览器的无刷新更新
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {}
})
//声明hotMiddleware无刷新更新的时机:html-webpack-plugin 的template更改之后
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

//将代理请求的配置应用到express服务上
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

//使用connect-history-api-fallback匹配资源
//如果不匹配就可以重定向到指定地址
app.use(require('connect-history-api-fallback')())

// 应用devMiddleware中间件
app.use(devMiddleware)
// 应用hotMiddleware中间件
app.use(hotMiddleware)

// 配置express静态资源目录
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static'))

var uri = 'http://localhost:' + port

//编译成功后打印uri
devMiddleware.waitUntilValid(function () {
  console.log('> Listening at ' + uri + '\n')
})
//启动express服务
module.exports = app.listen(port, function (err) {
  if (err) {
    console.log(err)
    return
  }
  // 满足条件则自动打开浏览器
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
})
复制代码
```

可见，webpack的编译，以及相对完善。我们也可以去优化一下对应的插件，比如：

```arduino
plugins: [
  new webpack.DefinePlugin({ // 编译时配置的全局变量
    'process.env': config.dev.env //当前环境为开发环境
  }),
  new webpack.HotModuleReplacementPlugin(), //热更新插件
  new webpack.NoEmitOnErrorPlugin(), //不触发错误,即编译后运行的包正常运行
  new HtmlWebpackPlugin({  //自动生成html文件,比如编译后文件的引入
    filename: 'index.html', //生成的文件名
    template: 'index.html', //模板
    inject: true
  }),
  new FriendlyErrorsPlugin() //友好的错误提示
]
复制代码
```

最后讲一下webpack的相关优化：

##### 构建速度的优化：

- 1.HappyPack 基于webpack的编译模式本是单线程，时间占时最多的Loader对文件的转换。开启HappyPack，可以讲任务分解成多个进程去并行处理。

  简单配置：

  new HappyPack({// 用唯一的标识符 id 来代表当前的 HappyPack 是用来处理一类特定的文件 id: 'babel',// 如何处理 .js 文件，用法和 Loader 配置中一样 loaders: ['babel-loader?cacheDirectory'],// ... 其它配置项 }),

  详细可参考：[www.fly63.com/nav/1472](https://link.juejin.cn?target=http%3A%2F%2Fwww.fly63.com%2Fnav%2F1472)

- 2.DllPlugin 可将一些Node_moudle一些编译好的库，常用而且不变的库，抽出来。这样就无需重新编译。

- 3.Loader 记录配置搜索范围，include，exclude 如：

  { test: /.js$/, //js文件加载器 exclude: /node_modules/, use: [ { loader: 'babel-loader?cacheDirectory=ture', options: { presets: ['@babel/preset-env'] }, include: Path2D.resolve(__dirname, 'src') } ] }

##### 优化打包大小：

- 1.tree shaking写法。(webpack 4 已自动引入) 即“摇树”。即只引入所需要引入部分，其余的代码讲会过滤。

- 2.压缩代码 当前最程愫的压缩工具是UglifyJS。HtmlWebpackPlugin也可配置minify等。

- 3.文件分离 多个文件加载，速度更快。 例如：mini-css-extract-plugin，将css独立出来。这样还有利于，部分“不变”的文件做缓存。

- 4.Scope Hoisting 开启后，分细分出模块直接的依赖关系，会自动帮我们合并函数。简单的配置：

  module,exports={ optimization:{     concatenateModules:true } }

#### 组件化

任何框架，团队都需要自己的组件化。（当然，有些团队，怕人员的流动性，全部不组件化，最简单的写法，笔者也遇过这种公司）。

一般来说，组件大致可以分为三类：

- 1)与业务无关的独立组件。
- 2)页面级别的组件。
- 3)业务上可复用的基础组件。

关于1），可以理解成现在的UI库（如element/vant），这里暂时不做独立组件分析。（晚些可能会写一篇如何写独立组件的文章，上传到npm。）

关于2），貌似当某一个模块，页面需要多次重复使用时候，就可以写成独立组件，这个貌似没什么好分析。

这里重点分析一下：** 3）业务上可复用的基础组件 ** 。

笔者写过的vue项目，都基本会封装20~30个业务通用组件。例如截图的my-form，my-table。如下： ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/9/173344c5e4a95849~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.awebp)

这里我以为myTable

emelent的table插件，的确已经很强大了。但是笔者虽然用上了emelent ui,但是业务代码却没有任何emelent的东西。

如果有一天，公司不再喜欢element ui的table，那so easy，我把我的mytable修改一下，所有页面即将同步。这就是组件化的魅力。

下边我以my-table为栗子，记录一下我组件化的要点： 1.合并封装分页，是表格不再关心分页问题。 2.统一全局表格样式（后期可随时修改） 3.业务脱离，使业务上无需再关心element的api如何定义，且可随时替换掉element。 4.自定义类型，本文提供select跟text控制，配置对象即可实现。 5.统一自定义缺省处理。 6.统一搜索按钮，搜索框。配置对象即可实现。

这些优势，以及对全局的拓展性，是不是比传统直接用的，有很大的优势？

当然，不好的地方，插件应该相对完善，考虑周全，需要一个全局统筹的人。对人员的流动的公司，的确很不友好。

下边是源码提供，可参考：

```xml
<template>
  <div>
    <h3 class="t_title">{{tName}}</h3>
    <div class="t_content">
      <el-form :inline="true" class="serach_form" >
        <el-form-item  v-for="(item, index) in tSerachList" :label="item.name" :key="index" v-if="tSerachList.length > 0 ">
          <div v-if="item.type == 'text'" >
            <el-input  :placeholder="item.name" v-model="tSerachList[index].value" ></el-input>
          </div>
          <div v-else-if="item.type == 'select'" >
            <el-select v-model="tSerachList[index].value"  :placeholder="item.name">
              <el-option  v-for="(cItem, cIndex) in item.list" :key="cIndex" :label="cItem.name" :value="cItem.value" ></el-option>
            </el-select>
          </div>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit" >查询</el-button>
        </el-form-item>
      </el-form>
      <!--按钮操作模块-->
      <el-row class="t_button_tab" v-for="(item,index) in tBtnOpeList" :key="index">
        <el-button  :type="item.type" @click="btnOpeHandle(item.opeList)" :render="item.render">{{item.label}}</el-button>
      </el-row>
    </div>

    <div class="t_table">
      <el-table
        :data="tableData"
        style="width: 100%">
        <el-table-column v-for="(item, index) in tableList" :key="index" v-bind="item">
          <template slot-scope="scope" >
            <my-table-render v-if="item.render" :row="scope.row" :render="item.render" ></my-table-render>
            <span  v-else>{{scope.row[item.key]}}</span>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="t_pagination">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="currentPage"
        :page-size="tPageSize"
        layout="prev, pager, next, jumper"
        :total="tTotal">
      </el-pagination>
    </div>
  </div>

</template>

<script>

  import MyTableRender from './my-table-render.vue'

  export default {
    props: {
      tTablecolumn: { //展示的列名
        type: Array,
        required: true
      },
      tName: { //页面的名称
        type: String,
          required: true
      },
      tUrl: { //请求的URL
        type: String,
        required: true
      },
      tParam: { //请求的额外参数
        type: Object,
        required: true
      },
      tSerachList: { //接口的额外数据
        type: Array,
        required: true
      },
      tBtnOpeList: {
        type: Array,
        required: false
      }
    },
    data () {
      return {
        arrea: "",
        currentPage: 1,
        tableData: [],
        tableList: [],
        tTotal: 0,
        tPageSize: 10,
        serachObj: {} //搜索的文本数据
      }
    },
    created () {
      this.getTableList()
      this.reloadTableList()
    },
    methods: {
      async getTableList () {
        var Obj = { pageNum: this.currentPage, pageSize: this.tPageSize }
        var that = this;
        var url = this.tUrl;
        var param = Object.assign(this.tParam, Obj, this.serachObj)
        const res = await this.utils.uGet({ url:url, query:param })
        var list = res.data.dataList
        that.tableData = list
        that.tTotal = res.data.total
      },
      // 提交
      reloadTableList () {
        var tableList = this.tTablecolumn
        for (var i = 0; i < tableList.length; i++) {
          tableList[i].prop = tableList[i].key
          tableList[i].label = tableList[i].name
        }
        this.tableList = tableList
      },
      onSubmit ( res ) {
        var that = this;
        const status = this.$store.getters.getUserStatus;
        if( status == 4 ){
          this.utils.uErrorAlert("临时用户无权限哦");
        } else {
          this.utils.uLoading(800);
          var tSerachList = this.tSerachList;
          var obj = {}
          for (var i = 0; i < tSerachList.length; i++) {
            obj[ tSerachList[i].key ] = tSerachList[i].value;
          }
          this.serachObj = obj;
          this.currentPage = 1;
          this.getTableList();
        }
      },
      handleSizeChange () {
      },
      handleCurrentChange (obj) {
        this.currentPage = obj
        this.getTableList()
      },
      btnOpeHandle(params){
        const status = this.$store.getters.getUserStatus;
        if( status == 4 ){
          this.utils.uErrorAlert("临时用户无权限哦");
        } else {
          this.$emit('handleBtn', params);
        }
      }
    },
    components: {
      MyTableRender
    }
  }
</script>


<style  lang="scss">

  @import '@/assets/scss/element-variables.scss';
  .serach_form{
    background: $theme-light;
    text-align: left;
    padding-top: 18px;
    padding-left: 20px;
  }
  .t_title{
    /*float: left;*/
    /*padding: 20px;*/
    /*font-size: 23px;*/
    color:$theme;
    text-align: left;
    border-left: 3px solid $theme;
    padding-left: 5px;
  }

  .t_content{
    clear: both;
  }

  .t_table{
    clear: both;
    padding: 20px;
  }

  .t_pagination{
    margin-top: 20px;
    float: right;
    margin-right: 20px;
  }
  .t_button_tab{
    text-align: left;
    margin-top: 18px;
  }

</style>

复制代码
```

### mini项目源码

最后送上个人手写的mini版本vue源码：

```kotlin
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>wz手写vue源码</title>
</head>
<body>
  <div id="app" class="body" >
	<div class="b_header" >
		<img class="b_img" src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/7/10/173344e271bf85af~tplv-t2oaga2asx-image.image" /><span>wz手写vue源码</span>
	</div>
	<div class="b_content" >
		<div class="n_name" >姓名：{{name}}</div>
		<div class="box" >
			<span>年龄：{{age}}</span>
		</div>
		<div>{{content}}</div>
		<div>
			<input type="text" wz-model="content" placeholder="请输入自我介绍"  />
		</div>
		<div wz-html="htmlSpan" ></div>
		<button @click="changeName" >点击提示</button>
	</div>
  </div>
  <style>
  .body{
	 text-align: left;
	 width: 300px;
	 margin: 0 auto;
	 margin-top: 100px;
  }
  .body .b_header{
	display:  flex;
	justify-item: center;
	justify-content: center;
	align-items: center;
	align-content: center;
	margin-bottom: 20px;
  }
  .body .b_header span{
	font-size: 21px;
  }
  .body .b_img{
	display: inline-flex;
	width: 20px;
	height: 20px;
	align-item: center;
  }
  .body .b_content{
	
  }
  .body div{
	margin-top: 10px;
	min-height: 20px;
  }
  button{
	margin-top: 20px;
  }
  </style>
  <script src="./wzVue.js"></script>
  <script>
    const w = new wzVue({
      el: '#app',
      data: {
        "name": "加载中...",
        "age": '加载中...',
		"content": "我是一枚优秀的程序员",
		"htmlSpan": '<a href="http://wwww.zhuangweizhan.com">点击欢迎进入个人主页 </a>'
      },
      created() {
        setTimeout(() => {
          this.age = "25岁";
		  this.name = "weizhan";
        }, 800);
      },
	  methods: {
		changeName() {
			alert("欢迎进入个人主页: http://www.zhuangweizhan.com");
		}
	  }
    })
  </script>
</body>
</html>


// js文件
/*
	本代码来自weizhan
*/
class wzVue {
	constructor(options){
		this.$options = options;
		console.log("this.$options===" + JSON.stringify(this.$options) );
		this.$data = options.data;
		this.$el = options.el;
		this.observer( this.$data );//添加observer监听
		new wzCompile( options.el, this);//添加文档解析
		if ( options.created ) {
			options.created.call(this);
		}
	}
	
	observer( data ){//监听data数据，双向绑定
		if( !data || typeof(data) !== 'object'){
			return;
		}
		Object.keys(data).forEach(key => {//如果是对象进行解析
			this.observerSet(key, data, data[key]);//监听data对象
			this.proxyData(key);//本地代理服务
		});
	}
	
	observerSet( key, obj, value ){
		this.observer(key);
		const dep = new Dep();
		Object.defineProperty( obj, key, {
			get(){
				Dep.target && dep.addDep(Dep.target);
				return value;
			},
			set( newValue ){
				if (newValue === value) {
				  return;
				}
				value = newValue;
				//通知变化
				dep.notiyDep();
			}
		})
	}
	
	proxyData(key){
		Object.defineProperty( this, key, {
			get(){
				return this.$data[key];
			},
			set( newVal ){
				this.$data[key] = newVal;
			}
		})	
	}
	
}

//存储数据数组
class Dep{
	constructor(){
		this.deps = [];
	}
	
	addDep(dep){
		this.deps.push(dep);
	}
	
	notiyDep(){
		this.deps.forEach(dep => {
			dep.update();
		})
	}
}

//个人编译器
class wzCompile{
	constructor(el, vm){
		this.$el = document.querySelector(el);
		
		this.$vm = vm;
		if (this.$el) {
			this.$fragment = this.getNodeChirdren( this.$el );
			this.compile( this.$fragment);
			this.$el.appendChild(this.$fragment);
		}
	}
	
	getNodeChirdren(el){
		const frag = document.createDocumentFragment();
		
		let child;
		while( (child = el.firstChild )){
			frag.appendChild( child );
		}
		return frag;
	}
	
	compile( el ){
		const childNodes = el.childNodes;
		Array.from(childNodes).forEach( node => {
			if( node.nodeType == 1 ) {//1为元素节点
				const nodeAttrs = node.attributes;
				Array.from(nodeAttrs).forEach( attr => {
					const attrName = attr.name;//属性名称
					const attrVal = attr.value;//属性值
					if( attrName.slice(0,3) === 'wz-' ){
						var tagName = attrName.substring(3);
						switch( tagName ){
							case "model":
								this.wzDir_model( node, attrVal );
							break;
							case "html":
								this.wzDir_html( node, attrVal );
							break;
						}
					}
					if( attrName.slice(0,1) === '@'  ){
						var tagName = attrName.substring(1);
						this.wzDir_click( node, attrVal );
					}
				})
			} else if( node.nodeType == 2 ){//2为属性节点
				console.log("nodeType=====22");
			} else if( node.nodeType == 3 ){//3为文本节点
				this.compileText( node );
			}
			
			// 递归子节点
			if (node.childNodes && node.childNodes.length > 0) {
				this.compile(node);
			}
		})
	}
	
	wzDir_click(node, attrVal){
		var fn = this.$vm.$options.methods[attrVal];
		node.addEventListener( 'click', fn.bind(this.$vm));
	}
	
	wzDir_model( node, value ){
		const vm = this.$vm;
		this.updaterAll( 'model', node, node.value );
		node.addEventListener("input", e => {
		  vm[value] = e.target.value;
		});
	}
	
	wzDir_html( node, value ){
		this.updaterHtml( node, this.$vm[value] );
	}
	
	updaterHtml( node, value ){
		node.innerHTML = value;
	}
	
	compileText( node ){
		if( typeof( node.textContent ) !== 'string' ) {
			return "";
		}
		console.log("node.textContent===" + node.textContent  );
		const reg = /({{(.*)}})/;
		const reg2 = /[^/{/}]+/;
		const key = String((node.textContent).match(reg)).match(reg2);//获取监听的key
		this.updaterAll( 'text', node, key );
	}
	
	updaterAll( type, node, key ) {
		switch( type ){
			case 'text':
				if( key ){
					const updater = this.updateText;
					const initVal = node.textContent;//记录原文本第一次的数据
					updater( node, this.$vm[key], initVal);
					new Watcher( this.$vm, key, initVal, function( value, initVal ){
						updater( node, value, initVal  );
					});
				}
				break;
			case 'model':
				const updater = this.updateModel;
				new Watcher( this.$vm, key, null, function( value, initVal ){
					updater( node, value );
				});
				break;
		}
	}

	updateModel( node, value ){
		node.value = value;
	}
	
	updateText( node, value, initVal ){
		var reg = /{{(.*)}}/ig;
		var replaceStr = String( initVal.match(reg) );
		var result = initVal.replace(replaceStr, value );
		node.textContent = result;
	}
	
}

class Watcher{
	
	constructor( vm, key, initVal, cb ){
		this.vm = vm;
		this.key = key;
		this.cb = cb;
		this.initVal = initVal;
		Dep.target = this;
		this.vm[this.key];
		Dep.target = null;
	}
	
	update(){
		this.cb.call( this.vm, this.vm[this.key], this.initVal );
	}

}
复制代码
```

### 文章结尾

文章均为原创手写，写一篇原创上万字的文章，明白了笔者的不易。

如有错误希望指出。

后续，我会继续react的总结。



作者：逐步前行
链接：https://juejin.cn/post/6847902225138876424
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

