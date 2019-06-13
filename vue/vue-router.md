# vue-router 实现原理

## man.js
```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')

```

## router.js
```js
import Vue from 'vue'
import KRouter from './kkb-router'
// 实际执行的是install方法
Vue.use(KRouter)

// 路由基本的配置
export default new KRouter({
  routes: [
    {
      path: '/',
      component: Home,
      // 进入路由之前的生命周期
      beforeEnter(from, to, next) {
        // next执行才跳转
        console.log(`beforeEnter from ${from} to ${to}`)
        // 模拟异步
        setTimeout(() => {
          // 2秒之后再跳转
          // 做任何权限认证的事情
          next()
        }, 1000)
      }
    },
    {
      path: '/about',
      component: () => import(/* webpackChunkName: "about" */ './views/About.vue')
    }
  ]
})
```

## KRouter 源码

```js
// 路由入口
let Vue

class KRouter {
  static install(_Vue) {
    // 别的地方要使用Vue
    Vue = _Vue
    Vue.mixin({
      // Vue生命周期钩子函数
      beforeCreate() {
        Vue.prototype.$kkbrouter = '来了小老弟，我是路由'
        /**
         * 这里的this有两种，一种事通过new Vue出来的，一种是组件VueComponent出来的
         * 比如router-view，router-link
         * this.$options.router如果为true，说明 是new Vue({
            router,
            ...
           }) 出来的
         */
        if (this.$options.router) {
          // 这是入口
          // 启动路由
          Vue.prototype.$krouter = this.$options.router
          this.$options.router.init()
          // 有时候路由需要动态跳转
          // this.$router.push('/about')
        }
      }
    })
  }
  constructor(options) {
    this.$options = options
    this.routeMap = {}
    // 使用Vue的响应式机制，路由切换的时候，做一些响应
    this.app = new Vue({
      data: {
        // 默认根目录
        current: '/'
      }
    })
  }
  init() {
    // 启动整个路由
    // 由插件use负责启动就可以了
    // 1. 监听hashchange事件
    this.bindEvents()
    // 2. 处理路由表
    this.createRouteMap()
    // 3. 初始化组件 router-view 和router-link
    this.initComponent()
    // ？生命周期，路由守卫
  }
  initComponent() {
    // router-view
    Vue.component('router-view', {
      render: h => {
        const component = this.routeMap[this.app.current].component
        // 使用h新建一个虚拟dom
        return h(component)
      }
    })

    Vue.component('router-link', {
      // props:['to'],
      props: {
        to: String
      },
      render(h) {
        // h == createElement
        // h三个参数，
        // 组件名
        // 参数
        // 子元素
        return h(
          'a',
          {
            attrs: {
              href: '#' + this.to
            }
          },
          [this.$slots.default]
        )
      }
      // import vue/dist/vue.js
      // template最终也是转换成render来执行
      // 需要compile
      // template:"<a :href='to'><slot></slot></a>"
    })
  }
  createRouteMap() {
    this.$options.routes.forEach(item => {
      this.routeMap[item.path] = item
    })
  }

  bindEvents() {
    console.log('绑定事件')
    window.addEventListener('hashchange', this.onHashChange.bind(this), false)
    window.addEventListener('load', this.onHashChange.bind(this), false)
  }
  getHash() {
    return window.location.hash.slice(1) || '/'
  }
  push(url) {
    // hash模式直接复制
    window.location.hash = url
    // history模式 使用pushState
  }
  getFrom(e) {
    let from, to
    if (e.newURL) {
      // 这是一个hashchange
      from = e.oldURL.split('#')[1]
      to = e.newURL.split('#')[1]
    } else {
      // 这是一个第一次加载触发的
      from = ''
      to = this.getHash()
    }
    return { from, to }
  }
  onHashChange(e) {
    // console.log(e)+
    // 路由跳转马上开始
    // console.log('路由准备跳转')
    // 获取当前的哈希值
    let hash = this.getHash()
    let router = this.routeMap[hash]
    let { from, to } = this.getFrom(e)
    // 修改this.app.current 借用了vue的响应式机制
    // console.log('hash变了')
    if (router.beforeEnter) {
      // 有生命周期
      router.beforeEnter(from, to, () => {
        this.app.current = hash
      })
    } else {
      this.app.current = hash
    }
  }
}

export default KRouter
```
