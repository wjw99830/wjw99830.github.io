<small>最后编辑于2019-09-18</small>

# Composition API
***
+ ref （声明一个状态，通过.value访问值，另一个作用是作为ref的宿主对象，运行时通过xxx.value访问模板中的ref）
+ reactive （将一个对象转化为响应式对象，无需通过.value访问）
+ computed
+ watch
+ inject
+ lifecycles （新版本的生命周期和之前的有一点细微的区别）
基于Vue响应式系统的依赖收集方式，声明一个跨组件的依赖（数据）是非常自然的，一个依赖本身不与任何组件耦合，使用ref函数便可以在一个单独的模块中声明一个依赖以及其对应的可复用逻辑，在不考虑Time Travel的情况下可以简单地代替全局状态管理，虽然用法与React Hooks相似，但其内部实现完全不同，原因便是Vue组件基于Watcher而不是基于本身无状态的组件函数，setup在返回值中明确了所有的依赖，因此也不会有hooks那样不允许放在条件语句中的限制。</p>
正如一句话所说的`always take composition over inheritance`
继承的沙盒特性导致不能灵活地定义一个组件的逻辑，而组合就没有这样的困扰，无论是mixin还是extend，在一个应用开发的过程中都远没有直接组合各个逻辑函数来的方便与清晰，且由于语义更为清晰的setup函数，因此黑魔法this也不再需要。
举个例子：有多个组件依赖一个跨组件的count状态以及对其进行操作的一组可复用逻辑。
```javascript
// count.js
import { value } from 'vue'
export const count = value(0);
export function inc() {
  count.value++;
}
export function dec() {
  count.value--;
}
export function reset() {
  count.value = 0;
}

// counter.js 
import { createComponent } from 'vue'
import { count, inc, dec } from 'count.js'
const Inc = createComponent({
  setup() {
    return {
      count,
      inc,
    }
  },
  template: `&lt;button @click="inc"&gt;Click to increment: {{ count }}&lt;/button&gt;`
})
const Dec = createComponent({
  setup() {
    return {
      count,
      dec,
    }
  },
  template: `&lt;button @click="dec"&gt;Click to decrement: {{ count }}&lt;/button&gt;`
})
// Loading.js
function async Loading(loading, method) {
  return function () {
    loading.value = true;
    await method.apply(this, arguments);
    loading.value = false;
  }
}

```
上例中，组件Dec与Inc共享了一个count状态，没有任何多余的代码，直接用setup函数按照语义组装到组件中便可以使用，且所有逻辑都是可复用的，仅需一次声明。
当然，除了声明与状态耦合的可复用逻辑，也完全可以模拟方法装饰器来声明与状态解耦的可复用逻辑。例如，当需要给一个方法增加一个执行时loading的逻辑时：
```javascript
// loading-button.js
import { createComponent } from 'vue'
import { value } from 'vue'
const LoadingButton = createComponent({
  template: `&lt;button :class="{ loading: loading }" @click="fetchSomething"&gt;submit&lt;/button&gt;`,
  setup() {
    const loading = value(false);
    function async fetchSomething() {
      // do sth async...
    }
    return {
      fetchSomething: Loading(loading, fetchSomething)
    }
  }
})
```
利用这种方法也避免了装饰器语法不稳定的困扰。

# 实践
实现一个useCache函数，传入数据id作为storage中的key，返回一个依赖和一个update函数（用来同步更新本地存储中的数据）
实现一个useSearch函数，对传入的响应式对象进行代理，返回一个keyword依赖和一个computed，在computed中计算和关键字匹配的数据