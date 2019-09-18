<small>最后编辑于2019-05-09</small>

# NProgress in Vue
今天接到一个需求: 给vue单页面应用的每次路由加入nprogress的进度条, 大部分页面的mounted中包含异步的数据获取操作. 显然, 在路由钩子里直接加入`Nprogress.start()`与`Nprogress.done()`是不符合页面数据获取进度的, 那么如何获取vue页面的加载完毕的时机呢? 在本项目中, 异步获取初始数据的操作放在mounted钩子里, 因此需要等到mounted异步函数执行完毕才能执行`NProgress.done()`.
第一反应是用装饰器重写页面的mounted方法, 使其await原先的函数之后执行`NProgress.done()`, 但是对于每一个参与路由的页面, 都需要用一次装饰器, 很麻烦, 因此决定在路由器的after钩子里重写to路由的最后一个matched的页面组件的mounted钩子, 这里需要注意的是同名路由不会再次调用mounted, 因此当同名路由也需要进度条时, 需要在钩子里主动调用from路由中的instences的mounted, instances即当前页面的组件实例;