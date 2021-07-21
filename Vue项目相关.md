Vue中一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝  
axios.create()	是添加了自定义配置的新的axios,对axios请求进行二次封装		axios.interceptors 有两种，一是请求拦截，二是返回拦截。  
```
// 请求拦截器（在请求之前进行一些配置）
axios.interceptors.request.use(function(config){
  //比如是否需要设置 token
  config.headers.token='hello world'
  return config
})
// 响应了拦截器（在响应之后对数据进行一些处理）
axios.interceptors.response.use(res=>{
  let data=res.data
  //比如响应一些报错信息
  return data
})
```
```
// 事件中心传值
  eBus.$emit('addition', {})	eBus.$on('addition', param => {});
  :class="[status ? 'className' : '', '']"
  this.$nextTick( () => {});
```
cookie存储的数量和字符数量都有限制，只能存储几十个，不超4096左右个字符；Cookie具有不可跨域名性  
Cookie保存在客户端浏览器中，而Session保存在服务器上  
用户通过用户名和密码发送请求；服务端验证,返回生成的token 给客户端,同时给数据库和Redis里关联token和用户信息；  
服务端验证,返回生成的token 给客户端,  同时给数据库和Redis里关联token和用户信息；服务端查询Redis+数据库, 验证token并返回数据。	跨服务器调用
```
//  vue-cookies，在vue中安装及使用cookie
//  1.使用命令安装：npm install vue-cookies --save	2.main.js里面引用
import VueCookies from 'vue-cookies'
Vue.use(VueCookies)
this.$cookies.set('key',value);//增加cookie，返回 this
this.$cookies.get('key');//获取cookie，返回 value
this.$cookies.remove('key');//删除cookie，返回 false 或 true
this.$cookies.isKey('key');//查询cookie是否存在该key，返回 false 或 true
this.$cookies.keys();//查询已存在的所有cookie，返回数组
```
```
// directory:表示检索的目录	useSubdirectories：表示是否检索子文件夹	regExp:匹配文件的正则表达式,一般是文件名
// 常常用来在组件内引入多个组件   在main.js中引入大量公共组件
   require.context(directory,useSubdirectories,regExp);
//（创建出）一个 context，其中文件来自 test 目录，request 以 `.test.js` 结尾。
   require.context('./test', false, /\.test\.js$/);
//（创建出）一个 context，其中所有文件都来自父文件夹及其所有子级文件夹，request 以 `.stories.js` 结尾。
   require.context('../', true, /\.stories\.js$/);
/**
导出的方法有 3 个属性： resolve, keys, id。  
resolve 是一个函数，它返回请求被解析后得到的模块 id。  
keys 也是一个函数，它返回一个数组，由所有可能被上下文模块处理的请求组成。 
id 是上下文模块里面所包含的模块 id. 它可能在你使用 module.hot.accept 的时候被用到
*/
```
import xxx from:若from的来源是文件夹，那么在package.json存在且设置正确的情况下，会默认加载package.json；若不满足，则加载index.js；若不满足，则加载index.vue。 

在一些情况下，会有设置二级路由，但一级路由不需要 component 的特殊需求，父如果不写component 属性,子集的component也不会显示：  
解决办法：`component: {render: (e) => e("router-view")}`  
路由配置完成后, 就要使用 router-view 进行渲染了 (只要有子路由, 就要用它来渲染);  
路由懒加载：
```
// 路由的异步加载
{ path: '/home', name: 'home', component: resolve => require(['@/components/home'],resolve) }
// 路由的魔法注释  使用import import函数的懒加载
const Home = () => import(/* webpackChunkName: 'ImportFuncDemo' */ '@/components/home')
```
Vuex  state专门用来保存 共享的状态值	 mutations: 专门书写方法,用来更新 state 中的值  
`this.$store.dispatch()`与`this.$store.commit()`方法的区别总的来说他们只是存取方式的不同,两个方法都是传值给vuex的mutation改变state
commit: 同步操作  
  存储 `this.$store.commit('changeValue',name)`  
  取值 `this.$store.state.changeValue`  
dispatch: 异步操作  
  存储 `this.$store.dispatch('getlists',name)`  
  取值 `this.$store.getters.getlists`  
vue-cropper:vue裁剪组件
