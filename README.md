# umi-dva

umi+dva 应用框架

## umi介绍

umi，中文可发音为乌米，是一个可插拔的企业级 react 应用框架。umi 以路由为基础的，支持类 next.js 的约定式路由，以及各种进阶的路由功能，并以此进行功能扩展，比如支持路由级的按需加载。然后配以完善的插件体系，覆盖从源码到构建产物的每个生命周期，支持各种功能扩展和业务需求。

## 环境准备

1. 首先得有 node，并确保 node 版本是 8.10 或以上。
2. npm i yarn tyarn -g
3. yarn global add umi
  如果提示 umi: command not found，你需要将 yarn global bin 路径配置到环境变量中，方法如下：
  C:\Users\Administrator\AppData\Local\Yarn\bin
  复制上面的 global bin 的路径，添加到系统环境变量 PATH。
4. 先找个地方建个空目录。然后通过 umi g 创建一些页面，
  umi g 是 umi generate 的别名，可用于快速生成 component、page、layout 等，并且可在插件里被扩展，比如 umi-plugin-dva 里扩展了 dva:model，然后就可以通过 umi g dva:model foo 快速 dva 的 mode
5. umi dev
6. 构建执行 umi build
  构建产物默认生成到 ./dist 下
7. 本地验证 发布之前，可以通过 serve 做本地验证 yarn global add serve, serve ./dist

## 通过脚手架创建项目

umi 通过 create-umi 提供脚手架能力，包含：

project，通用项目脚手架，支持选择是否启用 TypeScript，以及 umi-plugin-react 包含的功能
ant-design-pro，仅包含 ant-design-pro 布局的脚手架，具体页面可通过 umi block 添加
block，区块脚手架
plugin，插件脚手架
library，依赖（组件）库脚手架，基于 umi-plugin-library

1. 新建目录 执行 yarn create umi
2. 然后安装依赖，yarn
3. yarn start 启动本地开发

## 目录及约定

.
├── dist/                          // 默认的 build 输出目录
├── mock/                          // mock 文件所在目录，基于 express
├── config/
    ├── config.js                  // umi 配置，同 .umirc.js，二选一
└── src/                           // 源码目录，可选
    ├── layouts/index.js           // 全局布局
    ├── pages/                     // 页面目录，里面的文件即路由
        ├── .umi/                  // dev 临时目录，需添加到 .gitignore
        ├── .umi-production/       // build 临时目录，会自动删除
        ├── document.ejs           // HTML 模板
        ├── 404.js                 // 404 页面
        ├── page1.js               // 页面 1，任意命名，导出 react 组件
        ├── page1.test.js          // 用例文件，umi test 会匹配所有 .test.js 和 .e2e.js 结尾的文件
        └── page2.js               // 页面 2，任意命名
    ├── global.css                 // 约定的全局样式文件，自动引入，也可以用 global.less
    ├── global.js                  // 可以在这里加入 polyfill
    ├── app.js                     // 运行时配置文件
├── .umirc.js                      // umi 配置，同 config/config.js，二选一
├── .env                           // 环境变量
└── package.json

src/pages
约定 pages 下所有的 js、jsx、ts 和 tsx 文件即路由

src/layouts/index.js
全局布局，在路由外面套的一层路由。

src/app.(js|ts)
运行时配置文件，可以在这里扩展运行时的能力，比如修改路由、修改 render 方法等。

.umirc.(js|ts) 和 config/config.(js|ts)
编译时配置文件，二选一，不可共存。

## 路由

umi 会根据 pages 目录自动生成路由配置。

### 约定式路由

基础路由
假设 pages 目录结构如下：
-pages/
  -users/
    index.js
    list.js
  -index.js
那么，umi 会自动生成路由配置如下：
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/', component: './pages/users/index.js' },
  { path: '/users/list', component: './pages/users/list.js' },
]
注意：若 .umirc.(ts|js) 或 config/config.(ts|js) 文件中对 router 进行了配置，约定式路由将失效、新添的页面不会自动被 umi 编译，umi 将使用配置式路由。

### 动态路由

umi 里约定，带 $ 前缀的目录或文件为动态路由。
比如以下目录结构：
-pages/
  $post/
    index.js
    comments.js
  users/
    $id.js
  -index.js
会生成路由配置如下：
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/:id', component: './pages/users/$id.js' },
  { path: '/:post/', component: './pages/$post/index.js' },
  { path: '/:post/comments', component: './pages/$post/comments.js' },
]

### 可选的动态路由

umi 里约定动态路由如果带 $ 后缀，则为可选动态路由。

-pages/
  -users/
    $id$.js
  -index.js

会生成路由配置如下：
[
  { path: '/': component: './pages/index.js' },
  { path: '/users/:id?': component: './pages/users/$id$.js' },
]

### 嵌套路由

umi 里约定目录下有 _layout.js 时会生成嵌套路由，以 _layout.js 为该目录的 layout 。
-pages/
  -users/
     _layout.js
     $id.js
     index.js
会生成路由配置如下：
[
  { path: '/users', component: './pages/users/_layout.js',
    routes: [
     { path: '/users/', component: './pages/users/index.js' },
     { path: '/users/:id', component: './pages/users/$id.js' },
   ],
  },
]

### 全局 layout

约定 src/layouts/index.js 为全局路由，返回一个 React 组件，通过 props.children 渲染子组件。
export default function(props) {
  return (
    <>
      <Header />
      { props.children }
      <Footer />
    </>
  );
}

### 不同的全局 layout

你可能需要针对不同路由输出不同的全局 layout，umi 不支持这样的配置，但你仍可以在 layouts/index.js 对 location.path 做区分，渲染不同的 layout 。
export default function(props) {
  if (props.location.pathname === '/login') {
    return <SimpleLayout>{ props.children }</SimpleLayout>
  }
  return (
    <>
      <Header />
      { props.children }
      <Footer />
    </>
  );
}

### 通过注释扩展路由

约定路由文件的首个注释如果包含 yaml 格式的配置，则会被用于扩展路由。

比如：
-pages/
   index.js
如果 pages/index.js 里包含：
/**
 * title: Index Page
 * Routes:
 *   - ./src/routes/a.js
 *   - ./src/routes/b.js
 */
则会生成路由配置：
[
  { path: '/', component: './index.js',
    title: 'Index Page',
    Routes: [ './src/routes/a.js', './src/routes/b.js' ],
  },
]

### 配置式路由

如果你倾向于使用配置式的路由，可以配置 .umirc.(ts|js) 或者 config/config.(ts|js) 配置文件中的 routes 属性，此配置项存在时则不会对 src/pages 目录做约定式的解析。
比如：
export default {
  routes: [
    { path: '/', component: './a' },
    { path: '/list', component: './b', Routes: ['./routes/PrivateRoute.js'] },
    { path: '/users', component: './users/_layout',
      routes: [
        { path: '/users/detail', component: './users/detail' },
        { path: '/users/:id', component: './users/id' }
      ]
    },
  ],
};
注意：
component 是相对于 src/pages 目录的

### 启用 Hash 路由

umi 默认是用的 Browser History，如果要用 Hash History，需配置：
export default {
  history: 'hash',
}

## 在页面间跳转

在 umi 里，页面之间跳转有两种方式：声明式和命令式。

### 声明式

基于 umi/link，通常作为 React 组件使用。
import Link from 'umi/link';
export default () => (
  <Link to="/list">Go to list page</Link>
);

### 命令式

基于 umi/router，通常在事件处理中被调用。
import router from 'umi/router';

function goToListPage() {
  router.push('/list');
}

## 配置

### 配置文件

umi 允许在 .umirc.js 或 config/config.js （二选一，.umirc.js 优先）中进行配置，支持 ES6 语法。

### .umirc.local.js

.umirc.local.js 是本地的配置文件，不要提交到 git，所以通常需要配置到 .gitignore。如果存在，会和 .umirc.js 合并后再返回。

### UMI_ENV

可以通过环境变量 UMI_ENV 区分不同环境来指定配置。

## HTML 模板

### 修改默认模板

新建 src/pages/document.ejs，umi 约定如果这个文件存在，会作为默认模板，内容上需要保证有 
<div id="root"></div>，
比如：
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Your App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>

### 配置模板

模板里可通过 context 来获取到 umi 提供的变量，context 包含：

* route，路由对象，包含 path、component 等
* config，用户配置信息
* publicPath2.1.2+，webpack 的 output.publicPath 配置
* env，环境变量，值为 development 或 production
* 其他在路由上通过 context 扩展的配置信息

比如输出变量，
<link rel="icon" type="image/x-icon" href="<%= context.publicPath %>favicon.png" />

比如条件判断，
<% if(context.env === 'production') { %>
  <h2>生产环境</h2>
<% } else {%>
  <h2>开发环境</h2>
<% } %>

## Module Processor

如下列出的模块在 umi 中都会被自动处理，所以开发者无需关心这些模块是如何被处理的，以及他们的 webpack 配置是怎样的

### Module list

* .js, .jsx, .mjs, .jsx, .json: 由 babel-loader 处理
* .ts: 由 ts-loader 处理
* .graphql, .gql: 由 graphql-tag/loader 处理
* .css, .less, .sass: 由 css-loader, postcss-loader, less-loader 处理
* .svg: 由 @svgr/core 处理。使用 umi，你可以用如下方式引入 svg

## 介绍 Umi UI

