<small>最后编辑于2019-06-19</small>

# Prototype
看到一篇名为<a href="https://zhuanlan.zhihu.com/p/66368644" target="_blank" title="【nodejs原理&源码赏析（3）】欣赏手术级的原型链加工艺术">Node.js源码赏析</a>的文章，内容为对Node.js中Worker类实现的解析，源码使用的是ES6之前的构造函数实现的一个面向对象编程中的类，其中在实现继承这一步，除了调用父类的构造函数继承实例属性以及通过Object.setPrototypeOf将父类的原型设置为子类实例原型的原型之外，还将构造函数本身的原型设置为父类，其目的是实现通过子类也可以访问到父类的静态方法和属性。

基于原型的编程中，应该清楚地区分一个类本身的原型以及这个类的实例的原型，前者是类的__proto__属性，也就是调用Object.getPrototypeOf得到的对象，后者是类的prototype属性，这两者是有本质区别的，但容易混淆，__proto__只是浏览器厂商私自实现的一个属性，并没有在标准中存在，因此获取一个对象的原型的方法应是调用Object.getPrototypeOf或是Reflect.getPrototypeOf，prototype属性的意义是一个类的实例的原型，而非这个类的原型。