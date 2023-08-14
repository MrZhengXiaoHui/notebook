# `vue2` 源码

> `UI = render(state)`: 状态state是输入，页面UI是输出，状态输入变化，页面输出也随之变化。这种特性叫数据驱动视图。 state和UI是用户定的，而不变的是render()，所以vue充当render的角色。 当vue发现state变化之后，经过一系列处理，最终反应在UI上。

## 对象和数组的变化侦测

> 变化侦测就是追踪状态，或者说数据的变化，一旦发生了变化，就要去更新视图。

```js
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

```

### `Object` 的变化侦测

### `Array` 的变化侦测
