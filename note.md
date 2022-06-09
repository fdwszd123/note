# 路由

***



### 1.路由懒加载

``` js
import(/*webpackChunkName:"home"*/ "../pages/home.vue")
//  /*webpackChunkName:"home"*/:分包的时候可以给定名称
```



### 2.动态添加路由

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

  

### 3.删除路由

```js
// 1.添加一个同名的路由给其他的组件
router.removeRoute('home')
// 3.调用addRoute方法后会返回一个方法，调用这个方法会删除这个路由
const removeRoute = router.addRoute(categoryRoute);
removeRoute()
```

### 4.路由的其他方法

```js
router.hasRoute() //判断是否有这个路由
router.getRoutes() //获取一个包含所有路由的数组
```

### 5.路由导航守卫

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

### 6.辅助函数

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


# nexttick

将一个回调推迟到下一个Dom更新后去执行，在更改了一些数据后等dom更新后立即使用它

# axios

+++++

## 一.默认配置

axios的默认配置可以配置默认的baseUrl，timeOut，headers等

```ts
axios.defaults.baseURL = 'http://httpbin.org'
axios.defaults.timeout =  10000

```

## 二. 拦截器

### 1.请求拦截器

```ts
axios.interceptors.request.use(
  (config) => {
    //请求成功拦截到的请求具体的参数
    return config
  },
  (err) => {
    //请求失败
    return err
  }
)
```



### 2.响应拦截器

```ts
axios.interceptors.response.use(
  (res) => {
    //拦截到服务器响应成功
    return res
  },
  (err) => {
    //拦截到服务器响应失败
    return err
  }
)
```



# TS

++++

## 一.定义变量

### 1.类型推导

默认情况下进行赋值会将赋值的值的类型给这个变量

## 二.变量类型

### 1.any

适用于变量类型不确定，或者是变量类型需要改变的时候

### 2.unknown

描述不确定的变量

unknown类型只能赋值给any或者是unknown类型

any可以赋值给任意类型

### 3.void

函数没有返回值

### 4.never

表示永远不会发生值的类型

```ts
//永远不会返回值
function foo(): never {
  while (true) {}
}
function error(): never {
  throw new Error();
}
```

### 5.tuple

元组类型：多种元素的组合

```ts
let tom: [string, number] = ['Tom', 25];
//当元组类型的元素越界的时候越界的元素会被限制为每个类型的联合类型
```



## 三.类型断言

### 1.as

当有时候ts无法获取具体的类型信息时，我们需要进行断言

### 2.非空类型断言

```ts
function foo(message?: string) {
    //!表示一定不是空
  console.log(message!.length);
}
foo("hello");
foo("fantasy");

```

### 3.可选链

当对象的属性不存在时会短路，直接返回undefined，如果存在才会继续执行

```ts
type Person = {
  name: string;
  friend?: {
    name: string;
    age?: number;
  };
};
const info: Person = {
  name: "fantasy",
  //   friend: {
  //     name: "jay",
  //   },
};
console.log(info.name);
console.log(info.friend?.name);//当friend属性有的时候去读取name属性，没有friend属性的时候直接返回undefined
```

### 4.!!和??

 !!是把其他类型转化为布尔类型，类似于Boolean()

??(空值合并操作符) 如果左侧有值就 赋值没有赋右边的默认值

## 三.字面量类型

字面量类型的意义就是必须结合联合类型

```ts
type Align = "left" | "right" | "top";
let align: Align = "top";
align = "right";
```

### 1.字面量推理

as const 可以将类型转化为具体的字面量类型

## 四.类型缩小

### 1.typeof

### 2. ===,==,!==,!=,switch

### 3.instance of

### 4.in

## 五.函数类型

### 1.函数作为参数时定义类型

```ts
function foo() {}
function fn(foo: () => void) {}
fn(foo);

```

### 2.定义常量时编写函数类型

```ts
type addFnType = (num1: number, num2: number) => number;
const add: addFnType = (a1: number, a2: number) => {
  return a1 + a2;
};

```

### 3.案例

```ts
const calc = (
  n1: number,
  n2: number,
  fn: (num1: number, num2: number) => number
) => {
  return fn(n1, n2);
};
const res = calc(10, 20, function (a: number, b: number) {
  return a + b;
});
console.log(res);
```

## 六.函数参数的可选类型

### 1.可选类型

可选类型必须写在必选类型的后面

```ts
function foo(x: number, y?: number) {
  return x + y;
}
foo(2);
```

### 2.参数的默认值

```ts
function foo(x: number, y: number = 100) {
  return x + y;
}
foo(2);

```

### 3.剩余参数

```ts
function add(...nums: number[]) {
  let num: number = 0;
  nums.forEach((ele) => {
    num += ele;
  });
  console.log(num);
}
add(1, 2, 3);

```

### 4.this的默认推导

```ts
const info = {
  name: "fantsy",
  sayHellow() {
    console.log(this.name + "hello");
  },
};
info.sayHellow();

```

### 5.this的不明确类型

```ts
//明确指定this的类型
type nameType = { name: string };
function sayHello(this: nameType) {
  console.log(this.name);
}
const info = {
  name: "fantasy",
  sayHello,
};
info.sayHello(); //fantasy
//直接调用需要用call绑定this
sayHello.call({ name: "jay" }); //jay

```

### 6.函数的重载

函数的名称相同但是参数不同的几个函数，就是函数的重载

```ts
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a: any, b: any) {
  return a + b;
}
console.log(add(1, 5));
console.log(add("fan", "tasy"));
```

## 七.类

### 1.类的继承

```ts
class Person {
  name: string = "";
  age: number = 0;
  eat() {
    console.log("eat");
  }
}
class Student extends Person {
  snum: number = 100001;
  study() {
    console.log("study");
  }
}
class Teacher extends Person {
  title: string = "教授";
  teach() {
    console.log("teach");
  }
}
let stu = new Student();
console.log(stu); //Student { name: '', age: 0, snum: 100001 }    继承了Person的name和age属性

```



### 2.super

super可以调用父类的构造方法

```ts
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  eat() {
    console.log("eat");
  }
}
class Student extends Person {
  snum: number;
  constructor(name: string, age: number, snum: number) {
    //super调用父类的构造方法，初始化了name和age
    super(name, age);
    this.snum = snum;
  }
  study() {
    console.log("study");
  }
}
class Teacher extends Person {
  title: string = "教授";
  teach() {
    console.log("teach");
  }
}
let stu = new Student("fantasy", 18, 10002);
console.log(stu); //Student { name: 'fantasy', age: 18, snum: 10002 }
	
```

### 3.方法重写 

```ts
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  eat() {
    console.log("person-eat");
  }
}
class Student extends Person {
  snum: number;
  constructor(name: string, age: number, snum: number) {
    //super调用父类的构造方法，初始化了name和age
    super(name, age);
    this.snum = snum;
  }
  study() {
    console.log("study");
  }
  //子类重写父类的方法
  eat() {
    //也可以同时执行父类的方法
    super.eat();
    console.log("stu-eat");
  } 
}
class Teacher extends Person {
  title: string = "教授";
  teach() {
    console.log("teach");
  }
}
let stu = new Student("fantasy", 18, 10002);
stu.eat();
console.log(stu); //Student { name: 'fantasy', age: 18, snum: 10002 }

```

### 4.多态

父类引用指向不同的子类对象,也就是不同的子类继承自同一个父类，

```ts
class Animal {
  action() {
    console.log("animal action");
  }
}
class Fish extends Animal {
  action() {
    console.log("fish swimming");
  }
}
class Dog extends Animal {
  action() {
    console.log("dog running");
  }
}
//传入的是Animal但是有不同的表现形式
function makeActions(animals: Animal[]) {
  animals.forEach((animal) => {
    animal.action();
  });
}
makeActions([new Fish(), new Dog()]);

```

### 5.类的成员修饰符

#### ①public

public是默认的属性，是在任何地方可见公有的属性和方法

#### ②private

私有的只有在类的内部访问

```ts
class Person {
  private name: string = "";
  getName() {
    //只能在内部访问
    return this.name;
  }
}
let p = new Person();
// console.log(p.name);  因为name是私有的所以外部不能访问

p.getName();
```

#### ③protected

在类内部和子类中可以访问

```ts
class Person {
  protected name: string = "fantasy";
}
class Student extends Person {
  getName() {
    return this.name;
  }
}
let stu = new Student();
console.log(stu);

```

#### ④readonly

readonly是可以在构造器中赋值，赋值之后不可修改，成员本身是不可修改的，但是如果成员是一个对象的话，对象的属性是可以修改的

```ts
class Person {
  protected name: string = "fantasy";
  readonly age: number;
  constructor(age: number) {
    this.age = age;
  }
}
let p = new Person(18);
// p.age=19  age不可修改
console.log(p);
```

#### ⑤getters/setters访问器

可以通过访问器get  set获取和修改成员属性

```ts
class Person {
  private _name: string;
  constructor(name: string) {
    this._name = name;
  }
  set name(newVal) {
    this._name = newVal;
  }
  get name() {
    return this._name;
  }
}
let p = new Person("fan");
console.log(p.name); //fan
p.name = "fantasy";
console.log(p); //Person { _name: 'fantasy' }
```

### 6.类的静态成员

静态成员不需要new出来对象来使用，直接通过类就可以使用

```ts
class Person {
  static birthday: string = "1998-10-10";
  static smile() {
    console.log(":):):)");
  }
}
console.log(Person.birthday);
console.log(Person.smile());

```

### 7.抽象类abstract

继承是多态的前提，父类本身并不需要对某些方法进行具体的实现，所以父类中定义的方法，可以为抽象方法。

①抽象函数必须存在于抽象类当中

②抽象类无法被实例化

③抽象类当中的抽象方法子类必须实现 除非子类也是一个抽象类

```ts
//计算面积可能接收不同的形状
function makeArea(shape: Shape) {
  return shape.getArea();
}

//抽象函数必须存在在抽象类中
abstract class Shape {
  //这里Shape的getArea是没实现的如果直接传new Shape的话也是错误的
  //   getArea() {}

  //抽象方法是没有函数体的
  abstract getArea();
}
// new Shape()  抽象类不能被实例化
//抽象类当中的抽象方法子类必须实现 

class Circle extends Shape {
  private r: number;
  constructor(r: number) {
    super();
    this.r = r;
  }
  getArea() {
    return this.r * this.r * Math.PI;
  }
}
class Square extends Shape {
  private width: number;
  private height: number;
  constructor(width: number, height: number) {
    super();
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
}
let c = makeArea(new Circle(2));
let s = makeArea(new Square(2, 4));
console.log(c, s);
```

### 8.类的类型

类也可以作为一种类型

```ts
class Person {
  name: string = "fantasy";
  eat() {}
}
const p1 = new Person(); //p1的类型是Person
const p2: Person = {
  name: "fan",
  eat() {},
};
```

## 八.接口

### 1.接口定义对象类型

ts中可以使用类型别名(type)声明对象类型 ，还可以用接口(interface)声明类型

```ts
interface IIfoType {
  readonly name: string;
  age: number;
  firend?: {
    name: string;
  };
}
let info: IIfoType = {
  name: "fantasy",
  age: 18,
  firend: {
    name: "jay",
  },
};
```

### 2.接口定义索引类型

```ts
interface IIndexLanguage {
  [index: number]: string;
}
const language: IIndexLanguage = {
  0: "html",
  1: "css",
  2: "js",
  3: "vue",
};

```

### 3.接口定义函数类型

``` js
interface IcalcFn {
  (a: number, b: number): number;
}

function calc(a: number, b: number, calcFn: IcalcFn) {
  return calcFn(a, b);
}
const add: IcalcFn = (a, b) => {
  return a + b;
};
let res = calc(1, 2, add);
console.log(res);

export {};

```

### 4.接口的继承

```ts
interface IFly {
  fly: () => void;
}
interface ISwim {
  swim: () => void;
}
interface IAction extends IFly, ISwim {
  run: () => void;
}
const actions: IAction = {
  fly() {},
  swim() {},
  run() {},
};
export {};

```

### 5.交叉类型

```ts
interface ITypeA {
  name: string;
  age: number;
}
interface ITypeB {
  sex: string;
}

type typeA = ITypeA & ITypeB;
// 交叉类型typeA
// {
//     name: string,
//     age: number,
//     sex:string
// }
const info: typeA = {
  name: "fantasy",
  age: 18,
  sex: "男",
};
export {};

```

### ①联合类型的交叉类型

```ts
type typeA = "a" | "b" | "v";
type typeB = "c" | "b" | "e";
type myType = typeA & typeB;
//交叉后的类型type='b'
const a: myType = "b";

```

### 6.接口的实现

继承：只实现单继承
实现：实现接口，可以实现多个接口

```ts
interface ISwimm {
  swimm: () => void;
}
interface IRun {
  run: () => void;
}
//类实现接口
class Animal {}

//继承：只实现单继承
//实现：实现接口，可以实现多个接口
class Fish extends Animal implements IRun, ISwimm {
  swimm() {
    console.log("swimming");
  }
  run() {
    console.log("running");
  }
}
class Chad implements ISwimm {
  swimm() {}
}
//所有实现了接口的类对应的对象都可以传入，
function swimming(swimm: ISwimm) {
  swimm.swimm();
}
//实现了接口的不同的类
swimming(new Fish());
swimming(new Chad());

swimming({ swimm: () => {} }); //字面量也可以直接传入

```

### 7.interface和type的区别

interface通常用来定义对象类型

interface可以重复的对某个接口来定义属性和方法

type定义的是别名，别名是不能重复的

```ts
interface IFoo {
  name: string;
}
interface IFoo {
  age: number;
}
//接口可以是同一个名字，会把所有属性合并
const info: IFoo = {
  name: "fantasy",
  age: 18,
};

```

### 8.字面量赋值

```ts
interface IPerson {
  name: string;
  age: number;
}
const info = {
  name: "fantasy",
  age: 18,
  sex: "男",
};
function foo(p: IPerson) {
  console.log(p.name);
  console.log(p.age);
  // console.log(p.sex);  这里不能取到sex属性
}
foo(info); //info的类型不符合IPerson的类型但是ts做了类型擦除不会报错

```

## 九.枚举

```TS
enum Direction {
  LEFT,
  RIGHT,
  TOP,
  BOTTOM,
}
```

## 十.泛型

泛型就是将类型进行参数化

### 1.类型参数化

在定义函数的时候不决定参数的类型，在调用函数的时候再传入参数的类型	

```ts
function foo<Type>(num: Type): Type {
  return num;
}
foo<number>(30);
foo<string>("fantasy");
foo<string[]>(["fan"]);
foo(30)//不传类型的话，会进行类型推导，默认推导的类型为字面量类型
```

### 2.泛型接口

```ts
interface IPerson<T> {
  name: T;
}
const p: IPerson<string> = {
  name: "fanatsy",
};

const p1: IPerson<number> = {
  name: 18,
};

```

### 2.泛型类

```ts
class Point<T> {
  x: T;
  y: T;
  constructor(x: T, y: T) {
    this.x = x;
    this.y = y;
  }
}
const p1: Point<string> = new Point("1", "2");
const p2: Point<number> = new Point(3, 4);

```

### 3.泛型的类型约束

```ts
interface ILength {
  length: number;
}
function getLength<T extends ILength>(arg: T) {
  console.log(arg.length);
}
getLength("123");
getLength(["123"]);
getLength({ length: 10 });
// getLength(123)   没有lenght属性

```

## 十一.模块化

### 1.命名空间

​	把模块再划分作用域

```ts
export namespace time {
  export function format() {}
}
export namespace price {
  export function format() {}
}
```

### 2.类型查找

```ts
declare module 'lodash'{}
declare let name: string;

```



