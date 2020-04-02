---
title: Nuxt i18n国际化
date: 2019-03-31 22:17:15
categories:
 - javascript
tags:
 - i18n
 - nuxt

---



# Nuxt.js集成国际化



##  前言

​	公司官网需求要加上国际化， 之前官网使用的是vue的 nuxt框架，然后generate生成的静态页面，看了一下，nuxt官方有集成i18n的示例就跟着弄了一下，期间也踩了一些坑。



<!-- more -->

## 准备工作

### 安装

​	<code>npm install vue-i18n --save</code>



### 在nuxt.config.js 中配置

​	

```javascript
  plugins: ['~/plugins/i18n.js'],

  router: {
    middleware: ['i18n']
  }
```

### 在plugins文件夹下新建i18n.js

​	

```javascript
import Vue from 'vue'
import VueI18n from 'vue-i18n'

Vue.use(VueI18n)

export default ({ app, store }) => {
  // Set i18n instance on app
  // This way we can use it in middleware and pages asyncData/fetch
  app.i18n = new VueI18n({
    locale: store.state.locale,
    fallbackLocale: 'cn',
    messages: {
      en: require('~/locales/en.json'),
      cn: require('~/locales/cn.json')
    }
  })

  app.i18n.path = link => {
    if (app.i18n.locale === app.i18n.fallbackLocale) {
      return `/${link}`
    }

    return `/${app.i18n.locale}/${link}`
  }
}

```

### 在middleware文件夹下新建 i18n.js

​	 

```javascript
export default function ({ isHMR, app, store, route, params, error, redirect }) {
  const defaultLocale = app.i18n.fallbackLocale
  // If middleware is called from hot module replacement, ignore it
  if (isHMR) return
  // Get locale from params
  // const locale = params.lang || defaultLocale
  const locale = params.lang || defaultLocale
  if (store.state.locales.indexOf(locale) === -1) {
    return error({ message: 'This page could not be found.', statusCode: 404 })
  }
  // Set locale
  store.commit('SET_LANG', locale)
  app.i18n.locale = store.state.locale
  // If route is /<defaultLocale>/... -> redirect to /...
  if (locale === defaultLocale && route.fullPath.indexOf('/' + defaultLocale) === 0) {
    const toReplace = '^/' + defaultLocale + (route.fullPath.indexOf('/' + defaultLocale + '/') === 0 ? '/' : '')
    const re = new RegExp(toReplace)
    return redirect(
      route.fullPath.replace(re, '/')
    )
  }
}

```

## 创建本地语言库

​	在根目录下新建文件夹locales， 然后在文件夹下添加你要翻译的语言的json文件 。

### 在store文件夹下新建index.js

​	

```javascript
export const state = () => ({
  locales: ['en', 'cn'],
  locale: 'cn'
})

export const mutations = {
  SET_LANG(state, locale) {
    if (state.locales.indexOf(locale) !== -1) {
      state.locale = locale
    }
  }
}

```

### 踩坑……

​	好了，照着官方把该加的都加了，这下就能实现国际化的切换了吧。 然后参考了几篇文章， 通过 <code>**this**.$i18n.locale = Langname</code> 或是

 <code>this.$store.commit('SET_LANG', lang)</code> 切换语言。 结果要不就是404， 要不就是一个页面切换了，然后换个页面，也就是切换了路由又变回原来的语言了。。。   



​	结果找了好久发现是路由的问题，主要是middleware中的这一段代码:

```javascript
const locale = params.lang || defaultLocale
  if (store.state.locales.indexOf(locale) === -1) {
    return error({ message: 'This page could not be found.', statusCode: 404 })
  }
  // Set locale
  store.commit('SET_LANG', locale)
  app.i18n.locale = store.state.locale
```

每次我们切换页面的时候都会调用这个中间件，然后会去判断路由中的params 如果lang参数有值就将语言设置为对应的值。

​	 比如我的地址是  "http://localhost/zn/" 那么中间件就会将当前的lang设置为zn  如果就是"http://localhost/" 就会设置为默认的值.

​	所以之前我们手动设置了store中的值，切换了路由就会变回原样是因为每次中间件都会把locale设置成默认值(因为我们路由中没有带参数)。

​	可是当我们路由带上参数后切出现找不到页面的情况，这是因为当前路径下，比如 "http://localhost/zn/" 找不到我们的页面。 这是我们就要用到nuxt提供给我们的一个 **动态路由**的功能。 

​	[动态路由](https://zh.nuxtjs.org/guide/routing#%E5%8A%A8%E6%80%81%E8%B7%AF%E7%94%B1)根据官方可如下配置，因为nuxt是根据page下目录生成路由，可如下组织目录:

![A8CB811A-9DC9-4BB8-8C19-9EFB4ADDA033](https://ws4.sinaimg.cn/large/006tKfTcgy1g1mdn9yi7fj30e40n6jsd.jpg)

_lang 就是动态路由下的参数项，这样不管是带不带参数都能找到页面了。