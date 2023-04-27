
# 校园疫情防控管理系统

## 简介

本系统是一个为加强校园疫情防控的管理平台，可实现用户健康情况信息的收集和管理，登录用户可分为管理员，教师，学生。首页是一个校园疫情防控情况的图表展示；管理员可实现用户信息的管理以及发布疫情通知，其余用户都可接受到此通知；教师可查看班级学生的健康表情况以及对学生的申请离校作出批准；学生端可申请外出，健康表填报等。

关键技术点：

1.用户权限的控制：

- 前端控制页面级的权限。首先前端会有一份路由表，它表示了每一个路由可访问的权限。当用户登录之后，通过 token 获取用户的 role ，动态根据用户的 role 算出其对应有权限的路由，再通过router.addRoutes动态挂载路由，实现不同权限的用户显示不同的用户菜单和限制其所能进入的页面。
- 后端则会验证每一个涉及请求的操作，验证其是否有该操作的权限，每一个后端请求不管是 get 还是 post 都会让前端在请求 header里面携带用户的 token，后端解析token获取type用户类型来验证用户是否有权限执行该操作。若没有权限则抛出一个对应的状态码，前端检测到该状态码，做出相对应的操作。详情见权限控制篇

2.表格的分页：

- 前端分页：一次性请求所有数据，然后前端对数据进行截取展示，在element-ui的el-table的数据中实现，代码：


```js
   // 前端分页算法
    setTableData() {
       return this.tableData.slice((this.currentPage - 1) * this.pageSize, this.currentPage * this.pageSize)
    }
```

-  后端分页：需要通过axios异步发送当前所在页数和每页显示条数发送给后端，后端根据条件通过mysql语句查询对应数据返回展示。部分代码：


```js
 deleteRow(index, rows) {
      // console.log(index, rows)
      removeClass({
        c_id: rows.c_id,
        searchClass: this.inputSearchClass,
        searchFlag: this.searchFlag,
        pageSize: this.pageSize,
        currPage: this.currentPage - 1
      }).then((response) => {
        this.successOpen(response.msg)
```

3.express 全局错误处理：

由于在express框架中错误处理中间件函数不能自动捕获Promise错误，会报一个 `UnhandledPromiseRejectionWarning` 全局错误，需要自己进行处理，提升开发体验。

- promise.then().catch(),来捕获promise错误，并通过next（）将错误发送给异常中间件中，就可以捕获到错误，从而方便用户作出错误处理，但此方法每个函数都要使用.catch（）


-   随着async/await 流行以及配合try/catch 也可以捕获错误。但是代码存在大量重复的try/catch。


- 通过express后引入require(‘express-async-errors’)的方式就可以在express中捕获错误。他的本质就是代替了在每个express路由方法中打补丁的方式，通过复写express中Layer#handle方法，把每个Routing Function的错误统一用 next(err) 传递给错误处理中间件函数中，对错误进行统一处理。


最后给出了自定义的全局异常处理中间件函数，配合boom插件 返回错误对应的状态码以及信息，方便我们进行错误的处理及排查。

```js
/**
 * 自定义路由异常处理中间件
 * 注意两点：
 * 第一，方法的参数不能减少
 * 第二，方法的必须放在路由最后
//  */
router.use((err, req, res, next) => {
  console.log(err)
  // token 失效的错误提示
  if (err.name && err.name === 'UnauthorizedError') {
    const { status = 401, message } = err
    new Result(null, 'Token验证失败', {
      error: status,
      errMsg: message
    }).jwtError(res.status(status)) // 动态改变返回的状态码
  } else {
    console.log(err)
    // message就是自己写的
    const msg = (err && err.message) || '系统错误'
    const statusCode = (err.output && err.output.statusCode) || 500
    const errorMsg = (err.output && err.output.payload && err.output.payload.error) || err.message
    new Result(null, msg, {
      error: statusCode,
      errorMsg
    }).fail(res.status(statusCode))
  }
})
```

## 技术栈

1.使用vue+element-ui+vuex+vue-router+axios+echart框架进行前端开发

2.使用nodejs+express+mysql框架进行后端开发

3.前后端分离开发

## 前端目录结构

```bash
├── build                      // 构建相关  
├── config                     // 配置相关
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 主题 字体等静态资源
│   ├── components             // 全局公用组件
│   ├── directive              // 全局指令
│   ├── filtres                // 全局 filter
│   ├── icons                  // 项目所有 svg icons
│   ├── router                 // 路由
│   ├── store                  // 全局 store管理
│   ├── styles                 // 全局样式
│   ├── utils                  // 全局公用方法
│   ├── views                  // 组件
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   └── permission.js          // 权限管理
├── .babelrc                   // babel-loader 配置
├── eslintrc.js                // eslint 配置项
├── .gitignore                 // git 忽略项
├── favicon.ico                // favicon图标
├── index.html                 // html模板
└── package.json               // package.json
```

## 后端目录结构

```bash
├── bin                        // 运行相关
├── dao                        // 数据操作
├── model                      // 数据库操作
├── public                 	   // 静态文件
├── components                 // 路由
├── directive                  // 公用方法
├── app.js                     // 入口
└── package.json               // package.json
```



## 功能


```bash
- 登录
  - 用户验证
  - 管理员端
  - 学生端
  - 教师端

- 权限验证
  - 页面权限
  - 指令权限
  - 权限配置
  
- 管理员端
  - 首页 防控情况图表显示
  - 人员管理 添加/删除/搜索/编辑
  - 添加班级 搜索/删除
  - 发布通知 发布/详情/移除
  
- 学生端
  - 首页 防控情况图表显示
  - 我的通知 查看/阅读
  - 健康填报 填表上报
  - 请假申请 分页获取/申请
  - 个人信息 获取/头像上传/密码修改
  
- 教师端
  - 首页 防控情况图表显示
  - 发布通知 发布通知 发布/详情/移除
  - 请假管理 分页获取/同意/拒绝
  - 健康查看 分页获取查看/选择班级
  - 个人信息 获取/头像上传/密码修改

- 错误页面
  - 401
  - 404
  
- 表格
  - 分页
  - 删除
  - 搜索
  - 添加
  - 动态
  - 拖拽

```

## 开发

```bash
 # 克隆项目

git clone  https://gitee.com/anyway1212/epidemic-prevention.git

# 前端

1. vue-prevention # 进入前端目录

2. npm install  # 安装依赖

# 后端
1. node-prevention # 进入后端目录

2. npm install # 安装依赖
 
3. # 需要解压redis，完成后点击redis-server.exe 用来临时存储上传的用户数据，同时在pbulic目录下新建upload文件夹临时存放文件

# 数据库添加
1. # 使用navicat添加创建一个数据库并将数据库文件夹中的表直接添加进去就可以，navicat破解教程和如何创建数据库网上可查。
2. # 进入node-prevention目录下的model文件夹的model.js文件，修改mysql连接配置 改为自己的数据库

# 启动
1. npm run dev # 前端目录
2. npm run start # 后端目录
```

## 项目效果预览

### 登录
![登录界面](https://images.gitee.com/uploads/images/2021/0707/174711_df2013bd_8173160.png "屏幕截图.png")

### 首页
![首页](https://images.gitee.com/uploads/images/2021/0707/174858_df800e36_8173160.png "屏幕截图.png")

### 个人中心
![个人中心](https://images.gitee.com/uploads/images/2021/0707/181630_d00b9818_8173160.png "屏幕截图.png")

### 管理员端
| 人员管理 ![人员管理](https://images.gitee.com/uploads/images/2021/0708/111857_ddf6f87a_8173160.png "屏幕截图.png")   | 修改用户信息 ![修改用户信息](https://images.gitee.com/uploads/images/2021/0707/175220_94d0f225_8173160.png "屏幕截图.png")  | 添加班级![添加班级](https://images.gitee.com/uploads/images/2021/0708/111946_c3b1fb36_8173160.png "屏幕截图.png")   |
|---|---|---|

| 发布通知 ![发布通知](https://images.gitee.com/uploads/images/2021/0708/112030_851adb88_8173160.png "屏幕截图.png")  | 通知详情![通知详情](https://images.gitee.com/uploads/images/2021/0707/180327_643493b8_8173160.png "屏幕截图.png")  |   |
|---|---|---|

### 学生端
| 我的通知![我的通知](https://images.gitee.com/uploads/images/2021/0708/112133_f457e42a_8173160.png "屏幕截图.png")  | 健康填报![健康填报](https://images.gitee.com/uploads/images/2021/0708/112348_9a810834_8173160.png "屏幕截图.png")  |健康表信息![健康表信息](https://images.gitee.com/uploads/images/2021/0707/180943_caba0370_8173160.png "屏幕截图.png")   | 
|---|---|---|

|请假申请![输入图片说明](https://images.gitee.com/uploads/images/2021/0708/112207_d3e66c05_8173160.png "屏幕截图.png")  | 个人信息![个人信息](https://images.gitee.com/uploads/images/2021/0707/181541_b02f85be_8173160.png "屏幕截图.png")  |   |
|---|---|---|

### 教师端
| 请假管理![请假管理](https://images.gitee.com/uploads/images/2021/0708/112444_a7caad88_8173160.png "屏幕截图.png")  | 健康查看![健康查看](https://images.gitee.com/uploads/images/2021/0708/112509_4895d3af_8173160.png "屏幕截图.png")  |
|---|---|


## 前端详细介绍

### 登录方式介绍：

常用的登录方式有两种：

1. Cookie + Session 登录：Cookie就是存储在客户端的一小段数据；Session是指服务器为某个会话开启的一段独特的存储空间

   - 用户首次登录某页面，输入密码和用户名
   - 服务器收到密码和用户名验证无误后，会创建一个存储用户身份数据的session，同时会有一个与其对应的sessionId通过Set-Cookie的方式返回给前端
   - 下次发送非登录请求的时候，会自动携带Cookie，服务器通过验证sessionID来识别用户的身份，

   该方式存在的问题：

   - 由于服务器端需要对接大量的客户端，也就需要存放大量的Session信息，这样会导致服务器压力过大
   - 如果服务器端是一个集群，为了同步登陆状态，需要将Session信息同步到每一个机器上，无形中增加了服务器端的维护成本
   - 由于SessionId存放在Cookie中，所以无法避免CSRF攻击。		

2. jwt登陆：

   - 用户首次登录某网页，输入密码和用户名
   - 服务器收到密码和用户名验证无误后，通过jwt创建Token，并返回给前端
   - 下次发送非登录请求的时候，携带上token，用于后端验证用户身份，从而获取用户信息

   相比第一种的登录方式，token的方式的优缺点是：

   - 服务器端不需要存放token,所以不会造成服务器的压力，即使是服务器集群，也不会增加维护成本
   - Token可以存放在前端任何地方，可以不用存放在Cookie中，提高了页面的安全性
   - Token下发之后，只要在生效时间之内，就一直有效，如果服务器端想收回此 Token 的权限，并不容易

Token的生成方[式详情请见](https://juejin.cn/post/6845166891393089544#heading-5)

在综合考虑下，本次项目选择Token进行登录验证

### 登录流程图：

![登录流程](https://images.gitee.com/uploads/images/2021/0707/173826_0c3acc9e_8173160.png "屏幕截图.png")

1. 当用户填写完账号和密码点击登录按钮之后post给服务端，看是否验证通过，通过的话后端通过jwt生成token返回给前端，并将token存储在vux和cookie中，这样就能保证刷新页面之后浏览器还能记住用户的登录状态。
2. 在点击登录按钮进入后台界面之前，会发送请求获取个人信息，此时需要通过axios拦截器自动将token添加到请求头中，后端验证token并解析token获取用户个人信息返回给前端，从而进入到后台首页，生成对应角色的菜单栏。

### 权限控制：

一套完成的权限控制：需要解决一下几个问题;

1. 切换用户后，权限发生变化，注册的路由也应该要变化，理想情况是删除已注册的动态路由，然后才重新追加新路由。
2. 刷新页面时，如果用户鉴权还通过，那么其权限所允许的页面应该还能继续访问
3. 登出系统，即用户退出，需要清除已注册路由

#### 具体实现：

1. 创建vue实例的时候将vue-router挂载，但此时vue-router只会挂载一些登录和一些公用的路由。
2. 当用户登录之后，获取到用户角色role，将role和路由表每个页面需要的权限作比较，生成最终用户可以访问的路由表。
3. **调用router.addRoutes  这个API实现动态添加可访问的路由**。
4. 使用vuex管理路由表，根据vuex中可访问的路由表来渲染侧边栏组件。
5. 用户退出登录时候，需要移除该用户动态生成的路由。

#### 路由处理逻辑流程图：
![路由处理](https://images.gitee.com/uploads/images/2021/0711/220422_9148b06f_8173160.png "屏幕截图.png")

##### router/index.js

1. 通过在动态路由中的meta标签添加role属性，来标注这个路由是属于哪个角色所能访问的页面。
2. **注意**这里动态添加的路由数组最后的统配符路由需要放在最后面。

##### src/permission.js

通过全局前置守卫router.beforeEach实现

访问某个路由之前,，从 cookie中获取token，之后分为两种情况：

1. token存在：说明登录按钮按下或者处于后台界面，去判断访问的路由是否为登录界面：
   - 是登陆界面的话，说明非正常操作，直接重定向到首页。只有通过退出按钮以及出现错误或者token失效才能到达登录界面
   - 不是登录界面的话，就在vuex里获取用户角色，角色存在，直接进入对应页面；角色不存在，则是界面刷新或者刚进入后台界面，之后发送请求给后端获取用户角色，返回存在vuex中，根据用户角色动态生成路由表，从而进去访问的页面。出现异常通过try  catch捕获并重置token，重定向到登录界面。
   - 注意这里 在使用**addRoutes**()这个API的时候，出现next()失效 界面出现白屏跳转不过去的现象，主要原因是next()的时候路由并没有完全添加完成。解决办法就是next(to),再次进去这个router.beforeEach钩子 ，此时token和role存在之后 直接进入next()来释放钩子，这样就能确保所有的路由都已经挂载完成，这样router.beforeEach会执行两次。
2. token不存在：说明处于登录界面 或者手残 把token删了。
   - 判断访问的路由是否在免登录的白名单中，是 直接访问
   - 不在白名单中，重定向到登录界面

#### 用户路由表生成流程图：

![路由表生成](https://images.gitee.com/uploads/images/2021/0711/220234_f4a31102_8173160.png "屏幕截图.png")

##### store/modules/permission

实现思路就是遍历整个待添加的路由对象数组，过滤出该用户角色的路由，并和通用路由组合在一起形成该用户可进入的路由表

- 路由分为：constantRoutes 和 asyncRoutes
- 用户登录系统的时候，会动态生成路由表，其中constantRoutes是必然包含的 ，asyncRoutes是会根据用户角色进行过滤
- asyncRoutes过滤的逻辑是看下路由中是否包含meta和meta.roles属性，如果没有该属性，这就是一个通用路由，不需要权限校验；包含roles属性则会判断用户的角色是否命中路由中的任意一个权限，如果命中，则将路由保存下来，如果不命中，则会直接将路由舍弃
- asyncRoutes处理完成之后，会和constantRoutes合并为一个新的路由对象，保存在vuex的permission/routes中
- 用户登录系统后，侧边栏会从vuex中获取state.permission.routes，根据该路由动态渲染用户菜单

为什么需要将新的路由保存在vuex中呢？？

因为控制台上打印路由实例`router`，可以看到其下有个`options`属性，里面有个`routes`属性。这个就是我们创建路由实例时的`routes`选项内容。我们以为通过`addRoutes`动态注册路由后，新注册的内容也会出现在这个属性里，但结果却是没有。

`$router.options.routes`的内容只会是在创建实例时生成，后面追加的不会出现在这里。这意味着，在这个版本下的`vue-router`你没法通过路由实例对象来获知当前已注册的所有路由。

所以想要使用新的路由，就需要自己添加到vuex中。

#### 侧边栏显示

![菜单栏](https://images.gitee.com/uploads/images/2021/0711/220447_b77475d7_8173160.png "屏幕截图.png")

##### layout/components/Sidebar/index

菜单栏组件，从vuex中获取用户的路由表进行遍历传递给子组件SidebarItem显示

##### layout/components/Sidebar/SidebarItem

**实现思路：接收到路由表中的路由，判断每个路由中的子路由个数：**

- **0个子路由：直接显示该路由的图标和标题**
- 1个子路由：显示子路由的标题和菜单，父路由不显示
- 大于1个的子路由：通过递归组件的方式，循环遍历子路由再重复上面的判断

这样实现了侧边栏显示的操作，而且支持无线嵌套路由，多级子菜单。

##### 侧边栏对应菜单高亮：

通过el-menud的default-active属性动态绑定实现，具体代码：

```javascript
:default-active="activeMenu"
activeMenu() {
      // 进入到哪个路由 对应菜单高亮
      const route = this.$route
      const { meta, path } = route
      // if set path, the sidebar will highlight the path you set
      if (meta.activeMenu) {
        // 自定义对应路由 高亮哪个菜单
        return meta.activeMenu
      }
      return path
```

##### 点击侧边栏 刷新当前路由

在使用spa单页面开发之前，大部分都是多页面后台，用户每次点击侧边栏都会重新请求这个页面，有些用户就养成了这种习惯。但是现在spa就不一样，用户点击当前菜单不会刷新当前路由，因为vue-router会拦截你的路由，它会判断页面当前的url并没有变化，所以不会触发任何钩子或者是组件的变化。

那有没有什么方法解决？

既然不改变当前的URL就不会刷新当前路由，那么我只要不断改变URL的query参数不就可以实现刷新路由了吗。。

解决方案：监听侧边栏每个link的click事件，每次点击都给router push一个不一样的query来确保会重新刷新view。

但此方案有一个弊端就是URL后面会跟着一串很难看的query，只要你能接受这种情况的话 就不是什么大问题。

```javascript
clickLink(path) {
  this.$router.push({
    path,
    query: {
      t: +new Date() // 保证每次点击路由的query项都是不一样的，确保会重新刷新view
    }
  })
}

```

#### 清除动态添加的路由

在退出登录的时候，需要移除之前用户动态添加的路由，清空权限信息。由于vue-router里没有特定的API可以实现删除已注册的路由，替换同名路由。所以需要自己取实现，

- 最简单的方法就是退出之后刷新界面，路由重新加载，也可以实现。

  缺点：

  - 要重新刷新页面，如果系统网站本身初始化加载很慢的话，用户体验极差
  - 如果做得系统权限比较复杂的话，就比如在用户里根据不同任务有不同的权限，就不能用这种方式，因为切换任务并不会要重新登录。

- 在网上查找相关方案，最终选择的方案是需要重新创建一个router的matcher对象来替换之前的matcher对象。具体代码，[详情可见](https://github.com/vuejs/vue-router/issues/1234)

```javascript
// 创建路由实例的函数
const createRouter = () => new Router({
  // mode: 'history', // require service support
  scrollBehavior: () => ({ y: 0 }),
  routes: constantRoutes
})
// 这是伴随vue app实例化的初始化路由实例
const router = createRouter()
/**
 * 重置注册的路由
 * 主要是为了通过addRoutes方法动态注入新路由时，避免重复注册相同name路由
 */
export function resetRouter() {
  // 在退出登录时被调用
  const newRouter = createRouter()
  // 解决关键 重置了路由映射关系
  router.matcher = newRouter.matcher // reset router
}
```

该方案的实现关键是router.matcher = newRouter.matcher，这句代码相当于重置了路由映射关系，移除已注册的路由映射关系，跟新的路由实例的映射一样。

实现原理就是：因为所有的vue-router注册的路由信息都是存放在matcher之中的，所以当我们想清空路由的时候，，只要新建一个空的router实例，将他的matcher重新赋值给我们之间定义的路由就可以了。

在退出系统的时候需要做的事情：

```javascript
 		// 清除vuex中的token
        commit('SET_TOKEN', '')
        // 清除vuex中的角色
        commit('SET_ROLES', [])
        // 清除cookie中的token
        removeToken()
        // 删除动态添加的路由
        resetRouter()
```

#### 其余权限控制方案

实现方案[具体可见。](https://github.com/pekonchan/Blog/issues/20)

### axios使用

Axios是一个基于promise网络请求库，作用于node.js和浏览器中。

因为每次发送请求都需要在请求头中携带token，所以在使用原始的axios方法都需要自行添加代码，这样会出现大量的重复代码，而且每次发送请求都需要手动定义异常处理，就显得很麻烦。这时候就需要自行封装一个axios请求。

实现方式：

- 使用axios.create创建一个axios的实例，配置baseURL，以及网络请求超时时间；
- 添加axios请求拦截器，为每次http请求头添加token，并实现异步捕获和自定义错误处理。
- 添加axios响应拦截器，通过后端返回的状态码执行相应操作，以及token失效和其余错误的捕获和自定义处理。

代码：在utils/request.js

```javascript
// create an axios instance 这个实例是一个方法
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
  // withCredentials: true, // send cookies when cross-domain requests
  timeout: 5000 // request timeout
})

// request interceptor 添加请求拦截器
service.interceptors.request.use(
  config => {
    // do something before request is sent
    // 在请求头中添加token
    if (store.getters.token) {
      // let each request carry token
      // jwt中间件 规定用这样token形式传给后端
      config.headers['Authorization'] = `Bearer ${getToken()}`
    }
    return config
  },
  error => {
    // do something with request error
    console.log(error) // for debug
    return Promise.reject(error)
  }
)

// response interceptor 添加响应拦截器
service.interceptors.response.use(
  /**
   * If you want to get http information such as headers or status
   * Please return  response => response
  */

  /**
   * Determine the request status by custom code
   * Here is just an example
   * You can also judge the status by HTTP Status Code
   */
  response => {
    // console.log(response)
    // 响应中的data是从后端获取到的数据对象
    const res = response.data
    // res对象 包含 code msg data
    const errMsg = res.msg || '请求失败'
    // 这里的code 是后端定义的 0是成功 -1 是返回错误的 -2是token失效
    if (res.code !== 0) {
      Message({
        message: errMsg,
        type: 'warning',
        duration: 5 * 1000
      })
      return Promise.reject(new Error(errMsg))
    } else {
      return res
    }
  },
  error => {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    console.log('reject', { error }) // for debug
    const { msg, code } = error.response.data
    if (code === -2) {
      // token失效
      MessageBox.confirm('Token 已失效, 是否重新登录', '确认退出', {
        confirmButtonText: '重新登录',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        store.dispatch('user/resetToken').then(() => {
          // 因为路由全局守卫的原因
          // beforeEach 判断没有token 退回到登录界面
          location.reload()
        })
      })
      return Promise.reject(error)
    } else {
      Message({
        message: msg || '请求失败',
        type: 'error',
        duration: 5 * 1000
      })
      return Promise.reject(error)
    }
  }
)
```

### Table使用

#### 动态表格

实现：从后端获取表格数据之后，监听多选框中绑定的数据，与提前设定好表格头选项通过数组的filter过滤，求出表格头显示的数组。

![动态表格](https://gitee.com/anyway1212/epidemic-prevention/raw/master/image/动态表格.gif)

代码：

```javascript
watch: {
    checkboxVal(valArr) {
      this.formThead = this.formTheadOptions.filter(i => valArr.indexOf(i) >= 0)
      this.key = this.key + 1// 为了保证table 每次都会重渲 In order to ensure the table will be re-rendered each time
    }
  },
```

#### 表格拖拽

使用[sortable实现](https://github.com/SortableJS/Sortable)

![表格拖拽](https://gitee.com/anyway1212/epidemic-prevention/raw/master/image/表格拖拽.gif)

#### 表格增删改查分页

表格数据的增删改查和分页，我都是通过前端发送相关请求给后端，后端在数据库中增删改查以及排好序之后分页传给前端，前端只需要负责展示就可以了。

![表格展示](https://gitee.com/anyway1212/epidemic-prevention/raw/master/image/表格.gif)

##### mysql分页查询优化

比如分页获取用户信息：LIMIT 子句可以被用于指定 SELECT 语句返回的记录数

- 第一个？参数指定第一个返回记录行的偏移量，注意从`0`开始
- 第二个？参数指定返回记录行的最大数目
- 如果只给定一个参数：它表示返回最大的记录行数目

**直接使用数据库提供的SQL语句**：

```mysql
sql = 'select * from user where type = 1 LIMIT ?,?'
```

**LIMIT原理**：

**这种分页查询方式会从数据库第一条记录开始扫描，所以当数据库的表中有上万条记录及以上的时候，越往后，查询速度越慢，而且查询的数据越多，也会拖慢总查询速度。**

假如你现在要查询的偏移量为100w，那么limit会扫描1000010行，然后丢弃前100w行数据，留下最后10行，返回给我们，所以说我们只需要控制扫描的行数，查询的速度自然就快了，那如何控制扫描的行数呢？

##### 优化方法:

**1.使用索引覆盖+子查询优化**：如果用id作为数据表的主键：这种方式先定位偏移位置的 id，然后往后查询，这种方式适用于 id 递增的情况。

```mysql
select * from user where type=1 limit 100000,1;

select id from user where type=1 limit 100000,1;

select * from user where type=1 and 
id>=(select id from user where type=1 limit 100000,1) 
limit 100;

select * from user where type=1 limit 100000,100;
```

4条语句的查询时间如下：

- 第1条语句：3674ms
- 第2条语句：1315ms
- 第3条语句：1327ms
- 第4条语句：3710ms

针对上面的查询需要注意：

- 比较第1条语句和第2条语句：使用 select id 代替 select * 速度增加了3倍。这是因为用id主键作为索引的结果
- 比较第2条语句和第3条语句：速度相差几十毫秒
- 比较第3条语句和第4条语句：得益于 select id 速度增加，第3条语句查询速度增加了3倍

这种方式相较于原始一般的查询方法，将会增快数倍。

这其实是利用了索引覆盖的如下好处：

- 索引文件不包含行数据的所有信息，故其大小远小于数据文件，因此可以减少大量的IO操作。
- 索引覆盖只需要扫描一次索引树，不需要回表扫描行数据，所以性能比回表查询要高。

2.**使用 id 限定优化**：这种方式假设数据表的id是**连续递增**的，则我们根据查询的页数和查询的记录数可以算出查询的id的范围，可以使用 id between and 来查询：

```mysql
select * from user where type=1 
and id between 1000000 and 1000100 limit 100;
```

查询时间：15ms 12ms 9ms

这种查询方式能够极大地优化查询速度，基本能够在几十毫秒之内完成。限制是只能使用于明确知道id的情况，不过一般建立表的时候，都会添加基本的id字段，这为分页查询带来很多便利。

还可以有另外一种写法：

```mysql
select * from user where id >= 1000001 limit 100;
```

**3.使用索引覆盖+连接查询优化**

这种优化方式跟 1原理一样。也是先在索引上进行分页查询，当找到 id 后，再统一通过 JOIN 关联查询得到最终需要的数据详情。

```mysql
select * from t_order a Join (select id from t_order order by id limit 0, 10) b ON a.id = b.id;		
select * from t_order a Join (select id from t_order order by id limit 10000, 10) b ON a.id = b.id;	
select * from t_order a Join (select id from t_order order by id limit 100000, 10) b ON a.id = b.id;	
select * from t_order a Join (select id from t_order order by id limit 1000000, 10) b ON a.id = b.id;
select * from t_order a Join (select id from t_order order by id limit 10000000, 10) b ON a.id = b.id;

```

执行时间：

```
-- 0.001
-- 0.023
-- 0.028
-- 0.348
-- 2.955
```



#### 注意事项：

由于vue是一个MVVM框架，我们传统写代码是命令式编程，也就是说一步一步写清楚程序需要做什么（How to do What），拿到table这个dom之后就是命令式对dom增删改查。而我们现在用声明式编程，就是只需要告诉程序在哪些地方做什么就可以（Where to do What），只用关注data的变化就好了，我们这里的增删改查都是基于后端返回的tableData这个数组的。

但是如果需要对这个数组的数据进行修改，且使得vue能够检测到这个变动的数组并触发视图的更新，就不能使用数组索引的方式进行数组修改，比如：

```
vm.items[indexOfItem] = newValue
```

此方式数组虽元素然修改了，但是vue不会检测到，自然也就不会更新变化后的数据。这是因为js的限制。

解决方案：具体参考[vue官网]([列表渲染 — Vue.js (vuejs.org)](https://cn.vuejs.org/v2/guide/list.html#数组更新检测))

```js
// 数据元素修改
example1.items.splice(indexOfItem, 1, newValue)

//添加数据
this.list.unshift(this.temp);

//删除数据 
const index = this.list.indexOf(row); //找到要删除数据在list中的位置
this.list.splice(index, 1); //通过splice 删除数据

//修改数据
const index = this.list.indexOf(row); //找到修改的数据在list中的位置
this.list.splice(index, 1,this.updatedData); //通过splice 替换数据 触发视图更新


```

通过数组的方法对数组元素进行操作，vue就能检测到变化从而进行视图的更新。

### ECharts

[官网](https://echarts.apache.org/zh/index.html)

[webpack中使用ECharts文档](https://echarts.apache.org/zh/tutorial.html#%E5%9C%A8%E6%89%93%E5%8C%85%E7%8E%AF%E5%A2%83%E4%B8%AD%E4%BD%BF%E7%94%A8%20ECharts) [ECharts按需引入模块文档](https://echarts.apache.org/zh/tutorial.html#%E5%9C%A8%E6%89%93%E5%8C%85%E7%8E%AF%E5%A2%83%E4%B8%AD%E4%BD%BF%E7%94%A8%20ECharts) 接下来我们就要在vue中声明初始化ECharts了。因为ECharts初始化必须绑定dom，所以我们只能在vue的mounted生命周期里初始化。

示例：

```js
mounted() {
  this.initCharts();
},
methods: {
  this.initCharts() {
    this.chart = echarts.init(this.$el);
    this.setOptions();
  },
  setOptions() {
    this.chart.setOption({
      title: {
        text: 'ECharts 入门示例'
      },
      tooltip: {},
      xAxis: {
        data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
      },
      yAxis: {},
      series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
      }]
    })
  }
}

```

因为本项目中的data是从后端获取的，所以可以通过watch来触发setOptions方法

```js
//第一种 watch options变化 利用vue的深度 watcher，options一有变化就重新setOption
watch: {
  options: {
    handler(options) {
      this.chart.setOption(this.options)
    },
    deep: true
  },
}
//第二种 只watch 数据的变化 只有数据变化时触发ECharts
watch: {
  seriesData(val) {
    this.setOptions({series:val})
  }
}

```

### 跨域

本项目中使用的跨域方式是cors 全称为Cross Origin Resource Sharing(跨域资源共享),主要是在后端完成的。

每一次请求浏览器必须先以 OPTIONS 请求方式发送一个预检请求，从而获知服务器端对跨源请求所支持 HTTP 方法。在确认服务器允许该跨源请求的情况下，以实际的 HTTP 请求方法发送那个真正的请求。该方式好用的原因是只要第一次配好了，之后不管有多少接口和项目复用就可以了，一劳永逸的解决了跨域问题，而且不管是开发环境还是测试环境都能方便的使用。

但是想在前端配置的话，也有解决方案：

- 在dev开发模式下，使用webpack 的 proxy
- 在生产环境下，使用nginx反向代理

两种方案的原理都是一样的，通过搭建一个中转服务器来转发请求，从而获取资源数据，避免了跨域的问题。服务器之间没有跨域的问题，跨域只是限制的是浏览器，因为浏览器出于自身安全的考虑才有的同源策略。

### 路由懒加载

[路由懒加载 | Vue Router (vuejs.org):](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#把组件按组分块)当打包构建应用时，Javascript 包会变得非常大，影响页面加载速度。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

官方也就是在说，首先，路由通常会定义很多不同的页面，一般情况下，这个页面最后被打包在一个js文件夹中，但是页面这么多放在一个js文件中，必然会造成这个页面非常的大，如果我们一次性从服务器请求下来这个页面，可能需要花费一定的时间，而且可能会造成用户的电脑上出现短暂的空白情况。所以为了避免此情况的出现，使用路由懒加载。

懒加载的主要作用是：通过Webpack编译打包后，将路由对应的组件打包成一个个的js代码块，只有在这个路由被访问的时候，才会加载对应的组件。按需加载。

**而使用路由懒加载会出现一个问题就是**：

当我们的懒加载页面很多的话，在开发环境下，webpack热更新会很慢，导致修改代码之后重新运行项目会出现漫长的等待，猜测可能的原因是懒加载导致webpack每次的cache失效，所以每次的rebuild才会这么慢。

解决方案：原理就是不在开发环境使用懒加载，只有在正式环境下使用。

1. 封装一个_import（）的方法。具体实现：

   可以在路由设置的文件夹里增加一个开发环境路由配置和生产环境路由配置；

   ```
   // 在router中创建_import_development.js文件 开发环境 直接引用
   module.exports = file => require('../views/' + file + '.vue').default
   
   // 在router中创建_import_production.js文件 生产环境按需加载（通过函数）
   module.exports = file => () =>import('../views/' + file + '.vue')
   
   // 在router/index.js下 
   const _import = require('./_import_' + process.env.NODE_ENV);
   // in development env not use Lazy Loading,because Lazy Loading large page will cause webpack hot update too slow.so only in production use Lazy Loading
   
   const Login = _import('login/index');
   ```

   当使用 webpack 或 Browserify 类似的构建工具时，Vue 源码会根据 `process.env.NODE_ENV` 决定是否启用生产环境模式，默认情况为开发环境模式。在 webpack 与 Browserify 中都有方法来覆盖此变量，以启用 Vue 的生产环境模式，同时在构建过程中警告语句也会被压缩工具去除。所有这些在 `vue-cli` 模板中都预先配置好了，但了解一下怎样配置会更好。

   - `NODE_ENV` - 会是 `"development"`、`"production"` 或 `"test"` 中的一个。具体的值取决于应用运行的[模式](https://cli.vuejs.org/zh/guide/mode-and-env.html#模式)。

   - **模式**是 Vue CLI 项目中一个重要的概念。默认情况下，一个 Vue CLI 项目有三个模式：

     - `development` 模式用于 `vue-cli-service serve`
     - `test` 模式用于 `vue-cli-service test:unit`
     - `production` 模式用于 `vue-cli-service build` 和 `vue-cli-service test:e2e`

     根据启动的代码决定当前是在哪个模式下运行

     这样在router的index文件下通过require 导入对应的环境的路由文件，而怎么选择对应环境的路由文件是通过vue的脚手架是有三种模式的生产，测试，开发，我通过process.env.NODE_ENV环境变量就可以知道当前时处于哪种模式下，这样就可以找到对应环境的路由文件。

   该方案虽然能实现不在开发环境下使用路由懒加载，但是会出现`@/views/下的 .vue` 文件都会被打包。不管你是否被依赖。所以这样就产生了一个副作用，就是会多打包一些可能永远都用不到 js 代码。当然这只会增加 dist 文件的大小，但不会对线上代码产生任何的副作用

2. 使用`babel` 的 `plugins` [babel-plugin-dynamic-import-node](https://github.com/airbnb/babel-plugin-dynamic-import-node)。它只做一件事就是将所有的`import()`转化为`require()`，这样就可以用这个插件将所有异步组件都用同步的方式引入，并结合 [BABEL_ENV](https://babeljs.io/docs/usage/babelrc/#env-option) 这个`babel`环境变量，让它只作用于开发环境下，在开发环境中将所有`import()`转化为`require()`，这种方案解决了之前重复打包的问题，同时对代码的侵入性也很小，你平时写路由的时候只需要按照官方[文档](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)路由懒加载的方式就可以了，其它的都交给`babel`来处理，当你不想用这个方案的时候，也只要将它从`babel` 的 `plugins`中移除就可以了。

具体实现：

- 在命令行执行 `npm install babel-plugin-dynamic-import-node -S -D`
- 在`babel.config.js` 中添加插件

```js
module.exports = {
  presets: [
    // https://github.com/vuejs/vue-cli/tree/master/packages/@vue/babel-preset-app
    '@vue/cli-plugin-babel/preset'
  ],
  'env': {
    'development': {
      // babel-plugin-dynamic-import-node plugin only does one thing by converting all import() to require().
      // This plugin can significantly increase the speed of hot updates, when you have a large number of pages.
      // https://panjiachen.github.io/vue-element-admin-site/guide/advanced/lazy-loading.html
      'plugins': ['dynamic-import-node']
    }
  }
}

```

3.简单的方法是直接使用`webpack5`，它大幅提高了打包和编译速度，之后可能完全不需要搞这么复杂了，再多的页面热更新，都能很快，完全就不需要前面提到的解决方案了。

## 后端详细介绍

### 登录验证流程

  ![登录验证](https://images.gitee.com/uploads/images/2021/0711/221539_97f64f94_8173160.png "屏幕截图.png")
![登录验证2](https://images.gitee.com/uploads/images/2021/0711/221629_541fed53_8173160.png "屏幕截图.png")

### 跨域

本次项目中使用的cors进行跨域，这种方式需要发送两次http请求，第一次叫做OPTIONS请求。

关于为什么要发起 OPTIONS 请求，大家可以参考这篇文档：

[什么时候会发送options请求 (juejin.cn)](https://juejin.cn/post/6844903821634699277)

### 响应结果封装

通过使用同一封装返回的结果，来避免在写每个接口的时候都需要写上返回的结果，显得代码非常冗余，而且不易维护

```js
// 响应结果封装
const {
  CODE_ERROR, // -1
  CODE_SUCCESS, // 0
  CODE_TOKEN_EXPIRED,
  debug
} = require('../utils/constant')

// data 向前端返回的数据 msg信息 options一些辅助信息
class Result {
  constructor(data, msg = '操作成功', options) {
    this.data = null
    if (arguments.length === 0) {
      this.msg = '操作成功'
    } else if (arguments.length === 1) {
      this.msg = data
    } else {
      this.data = data
      this.msg = msg
      if (options) {
        this.options = options
      }
    }
  }

  createResult() {
    if (!this.code) {
      this.code = CODE_SUCCESS
    }
    let base = {
      code: this.code,
      msg: this.msg
    }
    if (this.data) {
      base.data = this.data
    }
    // options是一个对象
    if (this.options) {
      base = { ...base, ...this.options }
    }
    debug && console.log('response中的data:' + JSON.stringify(base))
    return base
  }

  json(res) {
    // 返回的base就是response中的data数据对象
    // 返回给前端{code, data, msg, options可选}
    res.json(this.createResult())
  }

  success(res) {
    // throw new Error('error....')
    this.code = CODE_SUCCESS
    this.json(res)
  }

  fail(res) {
    // throw new Error('error.....')
    this.code = CODE_ERROR
    this.json(res)
  }
  
  jwtError(res) {
    this.code = CODE_TOKEN_EXPIRED
    this.json(res)
  }
}

module.exports = Result

```

### JWT介绍

#### Token简介

##### Token 是什么

Token 本质是字符串，用于请求时附带在请求头中，校验请求是否合法及判断用户身份

##### Token 与 Session、Cookie 的区别

- Session 保存在服务端，用于客户端与服务端连接时，临时保存用户信息，当用户释放连接后，Session 将被释放；
- Cookie 保存在客户端，当客户端发起请求时，Cookie 会附带在 http header 中，提供给服务端辨识用户身份；
- Token 请求时提供，用于校验用户是否具备访问接口的权限。

##### Token 的用途

Token 的用途主要有三点：

- 拦截无效请求，降低服务器处理压力；
- 实现第三方 API 授权，无需每次都输入用户名密码鉴权；
- 身份校验，防止 CSRF 攻击。

#### JWT 简析

JSON Web Token（JWT）是非常流行的跨域身份验证解决方案。

一个jwt字符串分为三个部分：header（头信息），playload（消息体），signature（签名）

- header 部分指定了该 JWT 使用的签名算法: alg:表示加密算法，HS256是HMAC-SHA25的缩写，typ：token类型

  ```js
  header = '{"alg":"HS256","typ":"JWT"}'   // `HS256` 表示使用了 HMAC-SHA256 来生成签名。对称算法
  ```

- playload 部分表明了 JWT 的意图,存储的用户数据和令牌生成时间

  ```JS
  payload = '{"loggedInAs":"admin","iat":1422779638}'     //iat 表示令牌生成的时间
  ```

- signature 部分为 JWT 的签名，主要为了让 JWT 不能被随意篡改，签名的方法分为两个步骤：

1. 输入 `base64url` 编码的 header 部分、 `.` 、`base64url` 编码的 playload 部分，输出 unsignedToken。
2. 输入服务器端私钥、unsignedToken，输出 signature 签名。

```js
const base64Header = encodeBase64(header)
const base64Payload = encodeBase64(payload)
const unsignedToken = `${base64Header}.${base64Payload}`
const key = '服务器密钥'

signature = HMAC(key, unsignedToken)
```

最后的Token计算：

```js
const base64Header = encodeBase64(header)
const base64Payload = encodeBase64(payload)
const base64Signature = encodeBase64(signature)

token = `${base64Header}.${base64Payload}.${base64Signature}`

```

服务器在判断Token时：

```js
const [base64Header, base64Payload, base64Signature] = token.split('.')

const signature1 = decodeBase64(base64Signature)
const unsignedToken = `${base64Header}.${base64Payload}`
const signature2 = HMAC('服务器私钥', unsignedToken)

if(signature1 === signature2) {
  return '签名验证成功，token 没有被篡改'
}

const payload =  decodeBase64(base64Payload)
if(new Date() - payload.iat < 'token 有效期'){
  return 'token 有效'
}

```

有了 Token 之后，登录方式已经变得非常高效，