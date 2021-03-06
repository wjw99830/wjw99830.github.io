<small>最后编辑于2019-05-09</small>

# Web Components
目前Web Components的兼容性还不是很好, 只有现代主流浏览器的高版本才支持, 非Chromium的Edge还不支持, 但是作为标准, 未来它的使用将会越来越广泛, 还是有学习的必要的, 所以从写一个select组件入手来认识他的基本用法;

select样式参照Element UI的el-select;

根据MDN所示, 创建一个Web Components分为以下几步:
1. 创建一个ES6的Class, 继承HTMLElement;
2. 在构造函数中进行Shadow DOM的初始化, 完成一些初始化操作, 例如赋默认值以及事件绑定;
3. 为Shadow DOM添加style标签, 该style标签仅作用于Shadow DOM本身, 和外界完全隔绝, 效果类似Vue的scope;
4. 调用customElement.define注册自定义元素;

首先创建一个Class:
```javascript
class LktSelect extends HTMLElement {
  constructor() {
    super();
    // mode: open 使得js可以获取该自定义元素的shadow dom, 开发者工具中可见;
    const shadow = this.attachShadow({ mode: 'open' });
    const input = document.createElement('input');
    // 调用getAttribute获取自定义元素上的HTML特性;
    const placeholder = this.getAttribute('placeholder');
    const defaultValue = this.getAttribute('default');
    input.classList.add('lkt-select');
    input.placeholder = placeholder;
    input.value = defaultValue;
    input.readOnly = true;
    const style = document.createElement('style');
    style.textContent = ``; // 省略
    input.addEventListener('focus', (e) => {
      this.focus(e);
    });
    input.addEventListener('blur', () => {
      this.blur();
    });
    const globalStyle = document.createElement('style');
    globalStyle.textContent = ``; // 省略
    // globalStyle标签作用于整个页面;
    document.body.appendChild(globalStyle);
    shadow.appendChild(input);
    // 以下style标签仅作用于shadow dom;
    shadow.appendChild(style);
  }
}
```
调用customElements.define('lkt-select', LktSelect), 现在在HTML中可以直接使用lkt-select标签了;
```html
<lkt-select default="无选项" placeholder="请选择">
  <div>option 1</div>
</lkt-select>
```
自定义元素的子元素可以在对象中使用this.children访问到, 这里将其子元素克隆后放入一个容器追加到body后面作为options部分;
```javascript
renderOptions(top, left, height, width) {
  const container = document.createElement('div');
  // 使用fragment避免频繁地在真实DOM中进行插入操作;
  const fragment = document.createDocumentFragment();
  for (const option of this.children) {
    fragment.appendChild(option.cloneNode(true));
  }
  // 在真实DOM中仅有一次插入, 减少DOM操作的消耗;
  container.appendChild(fragment);
  container.classList.add('lkt-options');
  container.style.left = `${left}px`;
  container.style.top = `${top + height + 15}px`;
  container.style.width = `${width + 2}px`;
  // 事件委托完成select值的更新;
  container.addEventListener('click', (e) => {
    this.update(e.target.textContent);
  })
  return container;
}
```
Web Components的其中一部分内容就是template标签, 该标签内的内容不会被浏览器渲染, 但是js可以获取到其内容, 因此可以方便地定义一个自定义组件地基本结构, 可以使用slot插槽标签来作为占位符, 自定义元素的子元素指定插槽名后会被作为对应的插槽内容插入到模板中去实现灵活的渲染, Vue的slot也是参考了这一点;

值得一提的是, 在实现这个组件的过程中, 顺带实践了一下Vue的transition, 没读过Vue的具体实现, 但是核心应该离不开transitionend, 这里自己实现了一个:
```javascript
function transition(el, { enter, enterActive, enterTo, leave, leaveActive, leaveTo, isEnter }) {
  el.classList.add(isEnter ? enter : leave);
  el.classList.add(isEnter ? enterActive : leaveActive);
  requestAnimationFrame(() => {
    el.classList.add(isEnter ? enterTo : leaveTo)
    el.classList.remove(isEnter ? enter : leave);
  });
  function end(e) {
    if (isEnter) {
      e.target.classList.remove(enterActive);
    } else {
      e.target.removeEventListener('transitionend', end);
      e.target.remove();
    }
  }
  el.addEventListener('transitionend', end);
}
```
# 总结:
1. Web Components继承于HTMLElement, 可以使用它的所有方法;
2. 经测试发现在先解析HTML再加载script文件的情况下, 自定义元素仍然生效;
3. 若要使用, 必须调用customElement.define来定义一个标签和构造元素的类;
4. 实际使用时建议使用template标签来定义一个自定义元素的基本结构, 用DOM API来写比直接写Vue的h函数还麻烦;
5. 由于操作的是真实DOM, 没有了虚拟DOM及其diff的自动差值比较, 需要格外注重代码的性能表现, 否则便没有了使用原生js的意义;
6. 在自定义元素中操作外界dom时要格外注意避免污染外界dom;
7. shadow dom封装性过强, 没有必要将style标签全都放在shadow dom内;
8. 由于Vue的用户群体众多, 且Vue的模板写法是参照标准的, 因此Vue用户编写一个Web Components可能会更好上手;
9. 自定义元素本身不是shadow root, 可以将shadow root理解为该自定义元素的直接子元素, shadow root通过this.attachShadow创建并返回, 随后的访问应使用this.shadowRoot;
10. 使用自定义元素的children时, 应注意调用元素的cloneNode方法, 否则引起一些意料之外的表现, 比如将其子元素用于别处时, 删除操作会将自定义元素内的子元素也一并删除, 导致之后不再能够获取到子元素, 其原因是由于dom节点是引用类型, 同时也应尽量避免对大节点频繁执行cloneNode;
11. 想不到更多了, 之后再补充吧;