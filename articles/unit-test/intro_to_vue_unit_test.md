# Vue Unit Test Intro

## 初始化

1. 手动安装的步骤：安装 `@vue/cli`，使用 `vue create test-project` 命令初始化 Vue 项目，手动选择配置，在测试配置中选择 jest。
2. 测试文件写在 `test` 文件夹中，文件命名格式是 `*.spec.js`。
3. 运行测试的 script 一般是 `npm run test:unit`；如果想要监听文件修改自动执行测试代码，可以使用 `-- --watch` 参数，使用这个参数的前提是这是个 git 仓库。还有另外一个可以监听文件修改的参数 `-- --watchAll`。

## 渲染组件的两种方法：mount 和 shallowMount

这两个方法都会返回一个 wrapper 对象，这个对象包装了我们的 Vue 组件，以及一些操作组件的方法。

区别是：mount 方法会渲染整个组件，包括它的子组件；而 shallowMount 只会渲染当前组件，其子组件的位置会使用 component-stub 来替代。

> stub 是一个用来替换真实组件的“空对象”。

## 测试 props

通过 `mount` 和 `shallowMount` 的第二个参数给组件传递 props：

```js
const wrapper = mount(ComponentName, {
    propsData: {},
});
```

使用 factory 函数重构(DRY)，如果多个测试中都渲染了同一个组件，可以通过把组件对象的生成放到一个 factory 函数中：

```js
const msg = 'hello';
const factory = props => {
    return mount(Greeting, {
        propsData: {
            msg,
            ...props,
        },
    });
};
```

> 还可以使用 [setProps](https://vue-test-utils.vuejs.org/api/wrapper-array/#setprops-props)

## 测试 computed

computed 属性本质上就是 JS 函数而已。

语法：

```js
import TestComponent from '@/components/TestComponent';

expect(
    TestComponent.computed.computedName.call({
        propName: 'propValue',
    }),
);
```

computedName 就是组件中使用的 computed 属性。

call 方法传入一个 `this` 对象，替换 Vue 实例的 this。

除了上述方法，也可以选择传入 prop，渲染组件，然后测试 DOM 的内容。

**如何选择：call or mount?**

如果符合以下两种情况，可以使用 call：

-   组件的生命周期中可能有比较耗时的操作，而我们只是想要测试计算属性，可以使用 call 来避开执行生命周期中的代码。
-   希望使用一个自定义的 `this` 上下文。(wanna stub out some values on `this`)

## 模拟用户输入

-   使用 `setValue` 来给 `<input>` 元素设置值

-   用 `find` 找到元素，再使用 `trigger` 方法触发事件，这个方法支持原生和自定义事件，以及修饰符。

-   如果要测试用户输入后页面是否按预期渲染，可能需要先调用 `wrapper.vm.$nextTick()`(异步)，避免竞态条件。

## 测试异步

如果是通过 `Vue.prototyp.$http` 将请求方法挂载的原型上的话，可以通过 `mount` 的第二个参数传入 `mocks: { $http: mockHttp }`，其中 `mockHttp` 是自己实现的请求方法，具体实现是直接返回一个 Promise，resolve 数据。

> 要模拟 `Vue.prototype` 上的方法，都可以通过 mocks 传入

不过这样的话，测试还是会在 Promise resolve 之前结束，所以测试会不通过。

要解决这个问题，可以安装 **`flush-promises`** 依赖包，这个包会立即 resolve 所有 pending 的 Promise。

有两件事是需要测试的：

-   请求的 url 是否正确
-   返回的数据是否符合预期

## 测试自定义事件

如果事件是在组件的方法中触发的，可以通过 `wrapper.vm.methodName()` 触发事件，然后通过 `wrapper.emitted()` 获取触发事件对象，`emitted()` 方法会返回一个对象，以触发的事件名为键，触发事件时的 payload 为值(是一个数组，事件触发多少次就有多少项)

如果不想 mount 组件，可以通过第二种方法来触发事件。

`ComponentName.methods.methodName.call({$emit})`

使用 `call` 来调用触发事件的方法，绑定 `this` 为一个自定义对象，对象中有自定义的 `$emit` 方法。

```js
const events = {};
const $emit = (event, ...arg) => {
    events[event] || (events[event] = []);
    events[event].push([...arg]);
};
Form.methods.emitEvent.call({ $emit });
Form.methods.emitEvent.call({ $emit });
```

这种方法可以在组件生命周期函数中有很耗时操作的时候用，避开那些耗时操作。

## Mocking 全局对象

模拟 `Vue.prototype` 上的属性，一般常见的是模拟 `$store`, `$router`, `$t`(i18n)。

在 mount 方法的第二个参数中指定 `mocks` 属性。

**设置 mock 默认值**

在测试文件中使用 config API

```js
import { config } from '@vue/test-utils';

config.mocks['mock'] = 'Default Mock Value';
```

也可以把这些配置写进 `jest.init.js` 文件中，这个文件的内容会在所有测试开始前被执行。这个文件需要放在项目根目录，而且要在 `package.json` 中的 `jest` 字段中加上

```json
{
    "jest": {
        "setupFiles": ["<rootDir>/jest.init.js"]
    }
}
```

## 组件存根 stubbing components

**为什么会需要？**

在给父组件写测试时，我们可能并不关心子组件的逻辑，但是直接使用 mount 渲染父组件时，所有子组件和后代组件都会被原样渲染，后代组件中的代码逻辑可能会影响测试代码的运行时间。

**怎么做？**

方法一：在调用 mount 方法时传入第二个参数，有两种写法，一是指定值为 `true`，此时子组件会被默认渲染成 `<child-component-stub>`，另一种写法是自定义 hmtl `<div class="stub"></div>`。

这样子组件的所有方法都会被替换成 dummy method，不会执行任何逻辑。但是我们还是可以通过 `find` 等方法来找到子组件。

```js
mount(ParentCompoent, {
    stubs: {
        ChildComponent: true,
    },
});
```

方法二：使用 `shallowMount`，使用 `shallowMount` 的时候所有子组件都是默认存根的。

建议：默认使用 `shallowMount`，真的有需要的时候才用 `mount`

## 寻找元素和组件

**find**

使用 find 方法，这个方法接受的参数和 querySelector 的参数格式一样

找到元素后，可以使用 `isVisible()` 方法来判断元素是否可见(`v-show`)，或者使用 `exists()` 方法判断元素是否存在 DOM 中(`v-if`)

**findComponent**

接受 Vue 组件作为参数，使用 `find(Component)` 或者 `find({name: 'ComponentName'})` 也可以查找组件，不过不推荐

**findAll**

查找多个组件
