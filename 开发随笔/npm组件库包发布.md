# 发布一个组件库包到 NPM

一起养成写作习惯！这是我参与「掘金日新计划 · 4 月更文挑战」的第 14 天，[点击查看活动详情](https://juejin.cn/post/7080800226365145118)。

## 前言

> 问题抛出：在拥有了一系列自己开发的组件后，如何将它们整合到一个包里，像 element-UI 一样方便地使用？又如何将自己的组件开源分享？

这里以一个简单的 h1 标签组件为例，让我们看看它是如何经历这个奇妙的旅程，从二次元降落到我们的 “V”头上的（文章末尾附测试源码链接）

1. 环境准备
2. 组件开发及导入导出处理
3. NPM 包的发布
4. 包的使用

我们以搭建 Vue2 组件库为例开始实现。先来看看效果叭！从熟悉的界面开始......

![image-20220425114029654](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251140099.png)

最后的组件使用效果

![image-20220425114335755](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251143799.png)

## 环境准备

我们写的是 Vue2 组件库，所以使用的是 脚手架 vue-cli 4.5.0 版本（大于 2.9.0 版本）快速搭建项目，新版脚手架自带的 UI 操作界面非常好用，谁用谁知道。具体怎么用可以查看其他博客，这里我们搭建的是一个 UI 组件库，需要的依赖暂时很少，推荐先作如下配置

![image-20220425114539019](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251145078.png)

## 组件开发及导入导出处理

基础项目搭建完成，进入我们的开发阶段

### 关于项目目录结构

先看看该项目的初始整体结构吧（慢慢来，咱不犯错！）

![image-20220425114642465](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251146489.png)

再预览一下成果发布后的文件结构

![image-20220425114719642](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251147666.png)

### 关于组件

> 组件这一块的结构设计，参考了 element-UI 的源码，我们来看看

在根目录新建一个 plugins 文件夹，再建立它的子文件夹 components 文件夹（所有的组件都放这里），每个组件再分别建立自己的文件夹。

扩展：同时我们可以再 components 文件夹同级下建立我们封装的指令（directives ）和过滤器（filters）等工具的文件夹

**组件代码文件**（testPart.vue）：

```html
<template>
  <div>
    <h1>HUOHUOIT</h1>
  </div>
</template>

<script>
  export default {
    // 这里千万不能漏也不能写错了 name
    name: "testPart",
  };
</script>

<style></style>
```

**组件内部 JS 文件**（index.js）：用于实现按需导入功能

```javascript
import testPart from "./testPart.vue";

testPart.install = (Vue) => Vue.component(testPart.name, testPart);

export default testPart;
```

**组件系统下的 JS 文件**  （index.js）：全局注册并导出组件

```javascript
import testPart from "./components/testPart/index";

// 组件
const components = [testPart];

// 指令
// const directives = [xxxx]
// 过滤器
// const filters = [xxxx]

// 定义 install 方法，Vue 作为参数
const install = (Vue) => {
  // 判断是否安装，安装过就不用继续执行
  if (install.installed) return;
  install.installed = true;
  // 遍历注册所有组件
  components.map((component) => Vue.component(component.name, component));

  // 遍历注册所有指令
  // directives.map(directives => Vue.use(directives))

  // 遍历过滤器
  // filters.map(filters => Vue.use(filters))
};

// 检测到 Vue 再执行
if (typeof window !== "undefined" && window.Vue) {
  install(window.Vue);
}

export default {
  install,
  // 所有组件，必须具有 install 方法才能使用 Vue.use()
  testPart,
};
```

## NPM 包的发布

组件这一块这里不做详细讲解，处理完毕，我们开始 NPM 包的发布准备工作。

先关注一下**webpack 的打包**：项目根目录下建立 vue.config.js 文件

```javascript
const path = require("path");
module.exports = {
  // 修改 pages 入口
  pages: {
    index: {
      entry: "src/main.js", // 入口
      template: "public/index.html", // 模板
      filename: "index.html", // 输出文件
    },
  },
  // 扩展 webpack 配置
  chainWebpack: (config) => {
    // @ 默认指向 src 目录
    // 新增一个 ~ 指向 plugins
    config.resolve.alias.set("~", path.resolve("plugins"));

    // 把 plugins 加入编译，因为新增的文件默认是不被 webpack 处理的
    config.module
      .rule("js")
      .include.add(/plugins/)
      .end()
      .use("babel")
      .loader("babel-loader")
      .tap((options) => {
        // 修改它的选项...
        return options;
      });
  },
};
```

**打包发布前我们可以忽略一些不必要的文件**（.gitignore 文件处理）：把包的一些测试文件过滤掉，最终打包只留下直接封装的文件，即 plugins 中封装的暴露组件

```bash
src/
plugins/
public/
.editorconfig
.eslintrc.js
vue.config.js
babel.config.js
_.map
_.html
```

> 接下来是 NPM 发布的重点：packages.json 配置修改

最终版一览（写了注释的都是新增或者修改项）：

```json
{
  "name": "test-components", // 发布到 NPM 上的包名（注：包名重复会发布失败）
  "version": "0.1.0", // 版本号（注：代码更新发布前需同步更新版本号，不然无法成功发布）
  "description": "组件提交测试", // 关于包的描述（便于使用者快速了解包的相关信息）
  "private": false, // 是否私有（注：这里要设为 false，才能发布到 NPM）
  "author": "huohuo", // 作者名称
  "license": "MIT", // 包遵循的开源协议（参考 element 设为 MIT）
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint",
    "lib": "vue-cli-service build --target lib --name test-components -dest lib plugins/index.js" // --target lib：指定打包目录 --name testhuo-components：打包后文件名字 -dest lib ：打包后文件夹名称 plugins/index.js：打包入口文件
  },
  "keywords": [
    // 项目以及 npm 搜索关键词（注：影响你被搜索到的概率哦）
    "huohuo 的 h1 标题"
  ],
  "main": "dist/test-components.umd.min.js", // 主文件入口（最重要，路径名不要填错了！！！）
  "dependencies": {
    "core-js": "^3.6.5",
    "vue": "^2.6.11",
    "vue-router": "^3.2.0"
  }
  // ......
}
```

> **发布前千万要记得先打包**：npm run lib（我做这个例子的时候，后续增加组件进行测试，就是因为发布前没有重新打包，导致~~~~我明明每一步每个代码都正确，本地测试无误，发布后放到新项目里报组件未注册等错误。硬是发布了 20 多个版本才发现这个错误。emmmmmm.......痛苦）
> 最后：npm publish

注意可能出现的一个报错：

```bash
npm ERR! [no_perms] Private mode enable, only admin can publish this module [no_perms] Private mode enable, only admin can publish this module: wqui
```

这时候要把我们的淘宝镜像更改为如下，再 npm publish

```bash
npm config set registry <http://registry.npmjs.org/>
```

大功告成！！！

## 组件包的使用

**First：下载安装**

```bash
npm i test-components
```

**Second：引入项目中（项目的 main.js 文件内）**

import 了什么（这里的 testPart 可以自定义命名，代表引入整个 UI 库），就看你 export default 了哪些组件

![image-20220425150649366](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251506426.png)

如果你只想引入部分组件（按需导入，如 Button, Select）

```javascript
import Vue from 'vue';
import { Button, Select } from 'test-components';
import App from './App.vue';

Vue.component(Button.name, Button);
Vue.component(Select.name, Select);
/\* 或写为

- Vue.use(Button)
- Vue.use(Select)
  \*/
```

**Third：这里方便测试直接放在 App.vue 中啦！**

![image-20220425150754192](https://pictures-1312013355.cos.ap-guangzhou.myqcloud.com/pictures/202204251507214.png)

## 其他

> 关于 npm 包发布的一些操作

**废弃一个包**：这是用户仍然可以下载安装此包，只是会提示用户~ <message>

```bash
npm deprecate <pkg>[@<version>] <message>
// npm deprecate test-components@0.1.0 '这个包包不要啦，我已经换新包包啦！'
```

**删除一个包**（彻底删除自己 npm 上发布的包）：

```bash
npm unpublish 包名 --force
```

**版本控制：**

一个办法是改变在 packages.json 文件中的  version （新版本号要高于旧版本号），然后重新发布

也可以命令式改变：

```bash
npm version <update_type>
// 补丁：patch 小改：minor 大改：major
// npm version patch v0.1.0
```

[测试源码参考](https://github.com/Qingxuan001/public-NPM-packages)

​
