> 好难啊！！！！！！！

**1. 确定安装好vue-cli、webpack和webpack-cli**

vue-cli安装：

`npm install vue-cli -g`

webpack安装：

`npm install webpack -g`

`npm install webpack-cli -g`

**2. 生成初始项目**

`vue init webpack demo`

这里要是卡在download template，就把npm、vue-cli、webpack这类东西都更新到最新版。

**3. 我用idea打开了生成的文件夹demo**

**4. 在demo目录下，安装各种模块**

`npm install vue-router --save-dev`

`npm install --save axios vue-axios`

`npm i element-ui -S`

`npm install sass-loader node-sass --save-dev`

`npm install`

在main.js中引用这两个模块

```javascript
import VueRouter from './router'
import axios from "axios"
import VueAxios from "vue-axios"

//显示声明使用
Vue.use(VueRouter);
Vue.use(VueAxios, axios);
```

**5. 配置好router，创建router/index.js**

```javascript
import Vue from 'vue'
import VueRouter from "vue-router"

Vue.use(VueRouter);

export default new Router({
  routes: [
      //各种路由配置
  ]
});
```

**6. 现在把没用的东西都删了**，比如HelloWorld.vue，logo.png之类的

**7. 可以在config/index.js中把默认的8080端口改了**

**8. 可以用`npm run dev`命令在本地发布项目**



然后现在就可以愉快地码了，感觉这种模块化的前端比较适合用idea来开发。。。



另外还有：

sass那里可能有坑

把package.json里的sass-loader版本改到7.3.1

```shell
//卸载 node-sass
npm uninstall node-sass
//然后安装最新版本（5.0之前）
cnpm install node-sass@4.14.1
npm install
```





