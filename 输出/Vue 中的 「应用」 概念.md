---
type: permanent
---
# 🌐 核心观点

在 Vue3 中, 可以通过 `createApp` 函数来创建一个应用实例, 应用实例可以看作一个中心枢纽, 用来统筹各种资源.

---

# 🔖 详细解释

## 1	核心概念

 Vue 中的应用用于统筹一下资源:
 
- **共享的资源**: 哪些组件, 插件是全局可用的;
	
- **全局配置**: 如全局的错误处理;
	
- **挂载目标**: 这一套 UI 要被挂在到页面的**哪个 HTML 元素**中.

## 2	应用实例的主要职责

### 2.1	配置全局资源

你不需要在每个组件里手动导入常用功能. 通过应用实例, 你可以统一下发指令:

```js
app.component('MyButton', MyButton); // 注册全局组件
app.directive('focus', focusDirective); // 注册全局指令
app.use(router); // 安装插件
app.config.errorHandler = (err) => {
  /* 处理错误 */
}
```

### 2.2	提供依赖注入

让应用内的所有组件都拿到同一个配置信息.

```js
app.provide('theme', 'dark');
```

### 2.3	掌控生命周期

控制着 UI 的 mount 和 unmount, 是整个声明式渲染树的根基.

## 3	createApp 函数

通过 [createApp](https://cn.vuejs.org/api/application#createapp) 函数可以创建一个新的 Vue 实例, 第一个参数是根组件, 第二个参数是传递给根组件的 props, 可选.

```ts
function createApp(rootComponent: Component, rootProps?: object): App
```

应用实例必须在调用了 `mount()` 方法后才会渲染出来, 这个方法接收一个容器参数, 可以是实际的 DOM 元素, 也可以是一个 CSS 选择器字符串.

---

# 📚 参考内容

