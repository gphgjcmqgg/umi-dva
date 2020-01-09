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

* 项目管理。集中式管理本地项目。
* 配置管理。umi / bigfish 常用项目配置。
* 任务管理。集成启动、构建、测试、代码规范检查、重新安装依赖等常用操作。

## ------------------------------------------------------------------

## Mock 数据

Mock 数据是前端开发过程中必不可少的一环，是分离前后端开发的关键链路。通过预先跟服务器端约定好的接口，模拟请求数据甚至逻辑，能够让前端开发独立自主，不会被服务端的开发所阻塞。

### 使用 umi 的 mock 功能

umi 里约定 mock 文件夹下的文件或者 page(s) 文件夹下的 _mock 文件即 mock 文件，文件导出接口定义，支持基于 require 动态分析的实时刷新，支持 ES6 语法，以及友好的出错提示，详情参看 mock-data。
export default {
  // 支持值为 Object 和 Array
  'GET /api/users': { users: [1, 2] },

  // GET POST 可省略
  '/api/users/1': { id: 1 },

  // 支持自定义函数，API 参考 express@4
  'POST /api/users/create': (req, res) => { res.end('OK'); },
};
当客户端（浏览器）发送请求，如：GET /api/users，那么本地启动的 umi dev 会跟此配置文件匹配请求路径以及方法，如果匹配到了，就会将请求通过配置处理，就可以像样例一样，你可以直接返回数据，也可以通过函数处理以及重定向到另一个服务器。

比如定义如下映射规则：
'GET /api/currentUser': {
  name: 'momo.zxy',
  avatar: imgMap.user,
  userid: '00000001',
  notifyCount: 12,
},

### 引入 Mock.js

Mock.js 是常用的辅助生成模拟数据的第三方库，当然你可以用你喜欢的任意库来结合 roadhog 构建数据模拟功能。

import mockjs from 'mockjs';

export default {
  // 使用 mockjs 等三方库
  'GET /api/tags': mockjs.mock({
    'list|100': [{ name: '@city', 'value|1-100': 50, 'type|0-2': 1 }],
  }),
};

### 添加跨域请求头

设置 response 的请求头即可：
'POST /api/users/create': (req, res) => {
  ...
  res.setHeader('Access-Control-Allow-Origin', '*');
  ...
},

### 如何模拟延迟

为了更加真实的模拟网络数据请求，往往需要模拟网络延迟时间。

#### 手动添加 setTimeout 模拟延迟

'POST /api/forms': (req, res) => {
  setTimeout(() => {
    res.send('Ok');
  }, 1000);
},

#### 使用插件模拟延迟

import { delay } from 'roadhog-api-doc';

const proxy = {
  'GET /api/project/notice': getNotice,
  'GET /api/activities': getActivities,
  'GET /api/rule': getRule,
  'GET /api/tags': mockjs.mock({
    'list|100': [{ name: '@city', 'value|1-100': 50, 'type|0-2': 1 }]
  }),
  'GET /api/fake_list': getFakeList,
  'GET /api/fake_chart_data': getFakeChartData,
  'GET /api/profile/basic': getProfileBasicData,
  'GET /api/profile/advanced': getProfileAdvancedData,
  'POST /api/register': (req, res) => {
    res.send({ status: 'ok' });
  },
  'GET /api/notices': getNotices,
};

// 调用 delay 函数，统一处理
export default delay(proxy, 1000);

### 动态 Mock 数据

如果你需要动态生成 Mock 数据，你应该通过函数进行处理，

// 静态的
'/api/random': Mock.mock({
  // 只随机一次
  'number|1-100': 100,
}),

// 动态的
'/api/random': (req, res) => {
  res.send(Mock.mock({
    // 每次请求均产生随机值
    'number|1-100': 100,
  }))
},

## Use umi with dva

自>= umi@2起，dva的整合可以直接通过 umi-plugin-react 来配置。

* 按目录约定注册 model，无需手动 app.model
* 文件名即 namespace，可以省去 model 导出的 namespace key
* 无需手写 router.js，交给 umi 处理，支持 model 和 component 的按需加载
* 内置 query-string 处理，无需再手动解码和编码
* 内置 dva-loading 和 dva-immer，其中 dva-immer 需通过配置开启
* 开箱即用，无需安装额外依赖，比如 dva、dva-loading、dva-immer、path-to-regexp、object-assign、react、react-dom 等

## 使用 umi dva

用 yarn 安装依赖
yarn add umi-plugin-react

然后在 .umirc.js 里配置插件：
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: true,
      },
    ]
  ],
};
推荐开启 dva-immer 以简化 reducer 编写，
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: {
          immer: true
        }
      }
    ],
  ],
};

### model 注册

model 分两类，一是全局 model，二是页面 model。全局 model 存于 /src/models/ 目录，所有页面都可引用；页面 model 不能被其他页面所引用。
规则如下：

* src/models/**/*.js 为 global model
* src/pages/**/models/**/*.js 为 page model
* global model 全量载入，page model 在 production 时按需载入，在 development 时全量载入
* page model 为 page js 所在路径下 models/**/*.js 的文件
* page model 会向上查找，比如 page js 为 pages/a/b.js，他的 page model 为 pages/a/b/models/**/*.js + pages/a/models/**/*.js，依次类推
* 约定 model.js 为单文件 model，解决只有一个 model 时不需要建 models 目录的问题，有 model.js 则不去找 models/**/*.js

### 配置及插件

在 src 目录下新建 app.js，内容如下：
export const dva = {
  config: {
    onError(e) {
      e.preventDefault();
      console.error(e.message);
    },
  },
  plugins: [
    require('dva-logger')(),
  ],
};

### url 变化了，但页面组件不刷新，是什么原因？

layouts/index.js 里如果用了 connect 传数据，需要用 umi/withRouter 高阶一下。
import withRouter from 'umi/withRouter';
export default withRouter(connect(mapStateToProps)(LayoutComponent));

### 如何禁用包括 component 和 models 的按需加载？

在 .umirc.js 里配置：
export default {
  plugins: [
    [
      'umi-plugin-react',
      {
        dva: {
          dynamicImport: undefined // 配置在dva里
        },
        dynamicImport: undefined   // 或者直接写在react插件的根配置，写在这里也会被继承到上面的dva配置里
      }
    ],
  ],
};

### 全局 layout 使用 connect 后路由切换后没有刷新？

需用 withRouter 包一下导出的 react 组件，注意顺序。
import withRouter from 'umi/withRouter';
export default withRouter(connect()(Layout));

## 按需加载

出于性能的考虑，我们会对模块和组件进行按需加载。

### 按需加载组件

通过 umi/dynamic 接口实现，比如：
import dynamic from 'umi/dynamic';

const delay = (timeout) => new Promise(resolve => setTimeout(resolve, timeout));
const App = dynamic({
  loader: async function() {
    await delay(/* 1s */1000);
    return () => <div>I will render after 1s</div>;
  },
});

### 按需加载模块

通过 import() 实现，比如：
import('g2').then(() => {
  // do something with g2
});

## 运行时配置

我们通过 .umirc.js 做编译时的配置，这能覆盖大量场景，但有一些却是编译时很难触及的。
比如：

* 在出错时显示个 message 提示用户
* 在加载和路由切换时显示个 loading
* 页面载入完成时请求后端，根据响应动态修改路由

### 配置方式

umi 约定 src 目录下的 app.js 为运行时的配置文件。

### 配置列表

