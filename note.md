# 路由

***



- ### 路由懒加载

  ``` js
  import(/*webpackChunkName:"home"*/ "../pages/home.vue")
  //  /*webpackChunkName:"home"*/:分包的时候可以给定名称
  ```

  

- ### 动态添加路由

   ```js
   const categoryRoute = {
     path: "/category",
     component: () => import("../pages/category.vue"),
   };
   
   const homeAboutRoute = {
     path: "homeAbout",
     component: () => import("../pages/homeAbout.vue"),
   };
   router.addRoute(categoryRoute); //添加到了最外层
   router.addRoute("home", homeAboutRoute); //添加到子集
   ```

  

- ### 删除路由

   ```js
   // 1.添加一个同名的路由给其他的组件
   router.removeRoute('home')
   // 3.调用addRoute方法后会返回一个方法，调用这个方法会删除这个路由
   const removeRoute = router.addRoute(categoryRoute);
   removeRoute()
   ```

+ ### 路由的其他方法

  ```js
  router.hasRoute() //判断是否有这个路由
  router.getRoutes() //获取一个包含所有路由的数组
  ```
  


+ ### 路由导航守卫

  1.  beforeEach 

  ```JS
  /*
   *返回值
   *1.false 不进行导航
   *2.undefined或者不返回：进行默认导航
   *3.字符串（路径）：跳转到对应的路径
   *4.对象：类似于{path:"/home",query:{....}}
   */
  router.beforeEach((to, from) => {
    return false;
  });
  ```

  2. 其他守卫

     [其他守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

     

  

# vuex状态管理

***

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式 + 库**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

### 1.State

state是数据源，

### 2.Mutation

Mutation是唯一可以更改state中数据的方法，每个mutation都对应一个字符串类型的事件类型和回调方法

```js
mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
store.commit('increment')//使用mutation
```

### 3.Getter

Getter相当于vuex的计算属性

```js
getters: {
    total(state, getters) {
      let total = 0;
      state.books.forEach((book) => {
        total += book.count * book.price * getters.discount;
      });
      return total;
    },
    discount() {
      return 0.8;
    },
  },
```



### 4.Action

Mutations里面不可以放异步操作，所有的异步操作只能放在Action里面

Action提交的是Mutations而不是直接改变状态

Action可以包含任何的异步操作

context是一个和store相同的实例对象，包含store所有的属性，包括state，mutations，等

```JS
actions: {
    getData(context) {
      setTimeout(() => {
        context.commit("add");
      }, 1000);
    },
  },
      
  store.dispatch('getData')//使用actions
```

如果想知道Action里面的异步操作是否成功完成可以在action里面直接返回一个promise，在需要的组件中调用

```js
actions: {
    getData(context) {
       return new Promise((reslove,reject)=>{
        if(){

        //成功的话
            reslove()//把promise的状态从进行中改为成功
        }else{
            //失败的话
            reject()//把promise的状态从进行中改为失败
        }
    })
    },
  },
 //在需要使用的组件当中
      const promise=store.dispatch('getData')
      promise.then(res=>{
          //成功
      }).catch(err=>{
          //失败
      })

```



### 5.Module

每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块.

```js
const moduleA = {
  namespaced:true,
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

在模块化的时候vuex会把getters，actions，mutations，合并到一起也就是说当我们调用这些的时候是不分模块的，要是我们想调用某一个模块的时候就得开启命名空间namespaced

### 辅助函数

1. useStore()

   ```JS
   import { useStore } from "vuex";
   const store=useStore()//返回一个store的实例
   ```

2. mapState()

   mapState()传入一个数组['name','age']或者一个对象{sName:(state)=>state.name}//可以把state中的属性name映射为sName

   mapState()返回一个对象{name:fn,age:fn}  fn是一个必须经过computed处理才能使用的函数

   ```JS
   //mapState()的使用
   import { mapState, useStore } from "vuex";
   import { computed } from "vue";
   export function useState(raw) {
     const store = useStore(); //userStore()返回一个store的实例
     const storeStateFns = mapState(raw); //mapState(['name','age','height'])返回一个{counter：fn(){},name：fn(){},}这个fn必须经过computed处理才能使用
     const state = {};
     Object.keys(storeStateFns).forEach((key) => {
       const fn = storeStateFns[key].bind({ $store: store }); //bind是给绑定一个this
       state[key] = computed(fn); //state对象的每个value现在才是一个值，而storeStateFns的每个value是一个fn
     });
     return state;
   }
   useState(["name","age"])
   ```

   

2. mapGetters()

   ```js
   import { computed } from "vue";
   import { mapGetters, useStore } from "vuex";
   export function useGetters(raw) {
     const store = useStore();
     const gettersFn = mapGetters(raw);
     const getters = {};
     Object.keys(gettersFn).forEach((ele) => {
       const fn = gettersFn[ele].bind({ $store: store });
       getters[ele] = computed(fn);
     });
     return getters;
   }
   
   ```
   
   
   
2. mapMutations()

   ```js
   import { mapMutations } from 'vuex'
   
   export default {
     // ...
     methods: {
       ...mapMutations([
         'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
   
         // `mapMutations` 也支持载荷：
         'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
       ]),
       ...mapMutations({
         add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
       })
     }
   ```
   
   
   
5. mapActions()

   ```js
   import { mapActions } from 'vuex'
   
   export default {
     // ...
     methods: {
       ...mapActions([
         'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
   
         // `mapActions` 也支持载荷：
         'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
       ]),
       ...mapActions({
         add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
       })
     }
   }
   ```

   当使用了module进行了模块化之后使用这些辅助函数

   ```js
   //第一种写法：
   //多传一个参数指定命名空间
   computed:{
       ...mapState('home',["name","age"])
       ...mapGetters('home',['dobuleCounter'])
   }
   methods:{
       ...mapMutations("home",["add"])
       ...mapActions("home",["get"])
   }
   //第二种写法
   //直接使用vuex提供的辅助函数
   import { createNamespacedHelpers } from "vuex";
   const{mapState,mapGetters,mapMutations,mapActions}=createNamespacedHelpers('home')
   computed:{
       ...mapState(["name","age"])
       ...mapGetters(['dobuleCounter'])
   }
   methods:{
       ...mapMutations(["add"])
       ...mapActions(["get"])
   }
   ```

   

   
