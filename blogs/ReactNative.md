<small>最后编辑于2019-09-12</small>

# React Native
## Webview
使用最新的react-native-webview， 使用injectJavaScript向web页面传递信息（injectedJavaScript属性注入的代码仅运行一次），结合echarts封装rn端的echarts组件， RN中postMessage方法已不被支持， web页面中应使用ReactNativeWebView.postMessage， 不再使用window.postMessage。
```jsx
<ECharts height={ 200 } options={ options }></ECharts>
```
## TODO
+ 代理echarts事件， 通过postMessage通知RN触发对应的组件事件