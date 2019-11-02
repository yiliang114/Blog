### vue-cli@2.x 项目迁移日志

虽然 `vue-cli@3` 早就已经巨普及了，新项目应该已经很少有人还有使用 `vue-cli@2.x` 。 但是对于一些稍微早些时候的 `vue` 项目，如果当时没有做一些优化、配置，随着 `webpack` `vue` 等包的升级，有一些配置已经不一样了，并且关于 `vue-cli@2.x` 项目的文档、博客也越来越少，如果遇到问题也许也会有麻烦，因此就想着把当前的 `vue-cli@2.x` 项目原地升级配置。

#### 项目结构

`vue-cli@2.x` 项目结构，其中红色圈出的部分是 `2.x` 版本才有的。

![image](https://user-images.githubusercontent.com/11473889/58965618-744c6380-87e3-11e9-8993-337261b5ccaf.png)

`vue-cli@3.x` 目录结构， 箭头指出的两个文件的作用几乎完全替代上面的 `build` , `config` 文件夹中的文件，以及根目录下的 postcss 和 babel 配置文件的作用。

![image](https://user-images.githubusercontent.com/11473889/58965778-b4134b00-87e3-11e9-9e87-a2238ced0079.png)

#### 迁移配置

1. 新建 `babel.config.js` 文件， 内容是

   ```
   module.exports = {
     presets: [
       '@vue/app'
     ]
   }
   ```

2) 新建 `vue.config.js` 文件，内容是:

   ```
   const isProduct = process.env.NODE_ENV === 'production';

   module.exports = {
     publicPath: isProduct ? 'xxx' : ''
   };
   ```

   需要注意的是，在 `vue-cli@2.x` 中我们设置的 `assetsPublicPath` 属性，在 `vue-cli@3` 中已经更名为 `publicPath`， 这个属性可以简单理解为打包出来的 js css img 文件在被 `index.html` 文件引入的时候添加的前缀。

3) 直接将 `static` 文件夹更名为 `public` ,并且将根目录中的 `index.html` 文件也拖进 `public`文件夹中。这里需要注意的是，原来我们放在 `static` 中的静态资源，在代码中引用的时候，路径可能会写为 `/static/img/xxx.jpg`， 移动到 `public` 文件夹中之后，需要删除 `static` 前缀，直接引用 `/img/xxx.jpg` 即可。

4) 直接将 `static` 文件夹中的资源全都拖进,新建 `public` 文件夹

5) 接下来改动比较多的就是 `package.json`中的依赖包了

   ```
   // 修改执行脚本
   "start": "npm run serve",
   "serve": "vue-cli-service serve",
   "build": "vue-cli-service build",
   "lint": "vue-cli-service lint",
   ```

   ```
   // 需要手动更新一下 vue 和 vue-template-compiler 的版本，如果版本没有对象 npm start 就不成功，但是也不是严格的版本号一致，具体没研究过对应关系，直接从 vue-cli@3 初始化项目中抄版本号即可
     "dependencies": {
   		...
       "vue": "^2.6.10",
       "vue-router": "^2.3.1",
       "vuex": "^2.3.1"
     },
     // 这里的 7 个依赖都必须有，并且因为 webpack 功能被内置到 @vue/cli-service 中去了，所以原来 devDependencies 中根打包相关的依赖包都可以删除了。
     "devDependencies": {
       "@vue/cli-plugin-babel": "^3.7.0",
       "@vue/cli-plugin-eslint": "^3.7.0",
       "@vue/cli-service": "^3.7.0",
       "babel-eslint": "^10.0.1",
       "eslint": "^5.16.0",
       "eslint-plugin-vue": "^5.0.0",
       "vue-template-compiler": "^2.5.21",
       ...
     }
   ```

   ![image](https://user-images.githubusercontent.com/11473889/58967189-4b799d80-87e6-11e9-8059-dbbcd6d1b266.png)

6) 删除原来的依赖，重新安装新的依赖

   ```
   rm -rf node_modules && cnpm i
   ```

7) 重新执行试试 `npm start`

   此时很可能会遇到一个问题：

   ```
   [Vue warn]: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.
   ```

   是因为 vue 当前被编译的版本不对，造成这个影响主要是因为 webpack 的配置被修改引起的。 随便依照能找到很多解决办法，但是最好的办法是按照 `vue-cli@3`

   ```
   // 将 main.js 中 new Vue 的参数修改一下形式

   // vue-cli@2.x
   new Vue({
     el: '#app',
     router,
     components: { App },
     template: '<App/>'
   })

   // 修改为 vue-cli@3 中默认的形式
   new Vue({
     router,
     render: h => h(App),
   }).$mount('#app')
   ```
