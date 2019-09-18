<small>最后编辑于2019-06-18</small>

# 记一次Vue视图更新出现的反直觉行为
[知乎问题链接](https://www.zhihu.com/question/329842077)
原问题是有一个wrapper组件接收一个VNode类型的属性来对该节点添加额外样式，在示例代码中的父组件是通过更高一层的父组件传入的$slot来传递给wrapper组件的，出现的问题是外部传入的input slot上的v-model不是双向绑定，父组件对应的value变更时，slot中的input的value没有相应更新。
复制了代码并给content和slotlist计算属性分别加了个watch，给使用wrapper组件的父组件加了个updated hook跑了一下，在父组件的状态改变时这两个watch都没有被触发，但是updated hook触发了，意味着父组件状态变更时子组件触发了更新的，但它的插槽并没有变化，这是这个bug可以观察到的具体细节。
父组件在更新时，子组件也会触发updated的hook，但是这里的更新是行为上的，组件出现了一个更新的行为，但实际上属于这个组件的状态（data）都没有发生变化（Vue的依赖追踪指的是那些在data的返回值里且不以$或_开头的属性），因此在状态更新时，因为子组件并没有重新渲染，而仅仅是触发了更新的hook，因此slot还是组件创建时传入的那几个（input的value还是一开始的value），对应的dom也是和一开始的一样没有任何变化，于是就出现了这种“没有更新”的情况。
综上所述，出现这个“bug”的原因是slot-list组件依赖的数据不是被vue注册到组件实例的响应式数据，而是那个非响应式的属性$slot（数据传递的某一环使用了非响应式的数据），因此表现有点反直觉。
代码：
```vue
// parent
<template>
  <div id="app">
    <embed src="test.pdf" width="100%" height="100%">
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import DemoChild from './components/demo-child.vue';

@Component({
  name: 'app',
  components: { DemoChild },
})
export default class App extends Vue {}
</script>

// child
<template>
  <ul>
    <slot-list v-for="(item, index) of list" :key="index" :content="item">&lt/slot-list>
  </ul>
</template>
<script>
import SlotList from './slot-list';
export default {
  name: 'demo-child',
  components: { SlotList },
  computed: {
    list() {
      return (this.$slots.default || []).filter(item => item.tag);
    },
  },
  watch: {
    list() {
      console.log('slots changed.')
    }
  },
  updated() {
    console.log('updated.')
  }
};
</script>

// slot-list
<script>
  import Vue from 'vue'
  export default {
    name: 'slot-list',
    props: {
      content: {
        type: Object,
        default: () => ({}),
      },
    },
    watch: {
      content() {
        console.log('content has been changed.');
      },
    },
    render(h) {
      return h('li', { class: 'border-item' }, [this.content]);
    },
  }
  </script>
```