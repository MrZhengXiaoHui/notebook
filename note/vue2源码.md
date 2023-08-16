# `vue2` 源码

> `UI = render(state)`: 状态state是输入，页面UI是输出，状态输入变化，页面输出也随之变化。这种特性叫数据驱动视图。 state和UI是用户定的，而不变的是render()，所以vue充当render的角色。 当vue发现state变化之后，经过一系列处理，最终反应在UI上。

## 对象和数组的变化侦测

> 变化侦测就是追踪状态，或者说数据的变化，一旦发生了变化，就要去更新视图。

```js
// 本章用到的基础

/**
 * 直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象
 * @param {object} 需要定义属性的当前对象
 * @param {string} 当前需要定义的属性名
 * @param {object} 属性描述符
*/ 
Object.defineProperty(obj,'name',{
    value: 10, // 属性对应的值
    writable: false, // 控制该属性是否能被=赋值
    enumerable: false, // 控制该属性是否出现在对象的枚举属性中 Object.key()获取不到
    configurable: false, //控制该属性的除 value 和 writable 特性外的其他特性是否可以被修改和该属性能否被删除 如delete obj.name; delete没有删除成功也不会报错
    get() { // 属性的 getter 函数，当访问该属性时，会调用此函数
        return this.name
    },
    set(val) { // 属性的 setter 函数，当属性值被修改时，会调用此函数
        this.name = val
    }
})

/**
 * class对象
 */
class Point {
  constructor(x, y) {// 构造方法
    // this代表实例对象
    this.x = x;
    this.y = y;
  }
  // 自己定义的方法
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}

/**
 * 返回一个数组。成员是参数对象自身的（不含继承的）所有可遍历属性的键名
 */
Object.keys({ foo: 'bar', baz: 42 }) // ["foo", "baz"]

```

### `Object` 的变化侦测

+ 将对象的所有属性都转化成可观测对象

```js
// 源码位置：src/core/observer/index.js

/**
 * Observer类会通过递归的方式把一个对象的所有属性都转化成可观测对象
 */
export class Observer {
  constructor (value) {
    this.value = value
    // 给value新增一个__ob__属性，值为该value的Observer实例
    // 相当于为value打上标记，表示它已经被转化成响应式了，避免重复操作
    def(value,'__ob__',this)
    if (Array.isArray(value)) {
      // 当value为数组时的逻辑
      // ...
    } else {
      // 当value为对象时的逻辑  
      this.walk(value)
    }
  }
  // 遍历对象使对象转化成可观测对象
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
}
/**
 * 使一个对象转化成可观测对象
 * @param { Object } obj 对象
 * @param { String } key 对象的key
 * @param { Any } val 对象的某个key的值
 */
function defineReactive (obj,key,val) {
  // 如果只传了obj和key，那么val = obj[key]
  if (arguments.length === 2) {
    val = obj[key]
  }
  // 当val值还是为对象时，递归调用Observer方法使val值转为可观测对象
  if(typeof val === 'object'){
      new Observer(val)
  }
  //实例化一个依赖管理器，生成一个依赖管理数组dep
  const dep = new Dep()  

  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get(){
      dep.depend(); // 在getter中收集依赖

      console.log(`${key}属性被读取了`);
      return val;
    },
    set(newVal){
      if(val === newVal){
          return
      }
      console.log(`${key}属性被修改了`);
      val = newVal;

      dep.notify(); // 在setter中通知依赖更新
    }
  })
}
```

+ 依赖收集
  + 在getter中收集依赖，在setter中通知依赖更新

```js
// 源码位置：src/core/observer/dep.js

/**
 * 依赖管理器，生成一个依赖管理数组dep
 */
export default class Dep {
  constructor () {
    this.subs = []
  }

  addSub (sub) {
    this.subs.push(sub)
  }
  // 删除一个依赖
  removeSub (sub) {
    remove(this.subs, sub)
  }
  // 添加一个依赖
  depend () {
    if (window.target) {
      this.addSub(window.target)
    }
  }
  // 通知所有依赖更新
  notify () {
    const subs = this.subs.slice() // slice方法返回一个新数组
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

/**
 * 从数组中删除该项
 */
export function remove (arr, item) {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

### `Array` 的变化侦测
