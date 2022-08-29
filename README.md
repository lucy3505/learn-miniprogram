```js
   "dependencies": {
    "@vant/weapp": "^1.10.4",
    "miniprogram-api-promise": "^1.0.4",
    "mobx-miniprogram": "^4.13.2",
    "mobx-miniprogram-bindings": "^1.2.1"
  }
```

### 1.useComponent--->vant

```js
// 全局使用组件的话：
// app.json 中
"usingComponents": {
"van-button": "@vant/weapp/button/index"
},

// 局部使用组件的话：
// 在组件的 json 中：
"usingComponents": {
"van-button": "@vant/weapp/button/index"
},
```

## 2.promise 使用

app.js 中

```js
import {promisifyAll, prommiseAll} from 'miniprogram-api-promise'

const wxp = wx.p = {}
//promisify all wx's api
promisifyAll(wx,wxp)

// #使用：
// 事件处理函数
async getinfo(){
const {data:res}=await wx.p.request({
method:"GET",
url:"https://www.escook.cn/api/get",
data:{name:"zs",age:20}
})
console.log(res)
},
```

# store 的使用

1. 最外层建立 store 文件夹

```js
//store/store.js 中
import { action, observable } from "mobx-miniprogram";
export const store = observable({
  //数据字段
  numA: 1,
  numB: 2,
  //计算属性
  get sum() {
    return this.numA + this.numB;
  },
  //actions方法,用来修改store中的数据
  updateNum1: action(function (step) {
    this.numA += step;
  }),
  updateNum2: action(function (step) {
    this.numB += step;
  }),
});
```

2.文件中的使用

```js
import {createStoreBindings} from "mobx-miniprogram-bindings"
import {store} from '../../store'
 onLoad() {
    this.storeBindings=createStoreBindings(this,{
        store,
        fields:["numA","numB","sum"],
        actions:["updateNum1"]
    })
  },
  btnHandler1(e){
    console.log(e)
    const step = e.target.dataset.step
    this.updateNum1(step)
  },
  onUnload:function(){
      this.storeBindings.destroyStoreBindings()
  }

```

```html
<van-button type="primary" bindtap="btnHandler1" data-step="{{1}}"
  >numA + 1</van-button
>
<van-button type="danger" data-step="{{-1}}" bindtap="btnHandler1"
  >numA - 1</van-button
>
```

# 将 Store 中的成员绑定到组件中

```js
import { storeBindingsBehavior } from "mobx-miniprogram-bindings";
import { store } from "../../store/store";

Component({
  behaviors: [storeBindingsBehavior], // 通过storeBindingsBehavior 来实现自动绑定
  storeBindings: {
    store, //指定要绑定的Store
    fields: {
      //指定要绑定的字段数据
      numA: () => store.numA, //绑定字段的第 1 种方式
      numB: (store) => store.numB, //绑定字段的第 2 种方式
      sum: sum, //绑定字段的第 3 种方式
    },
    actions: {
      //指定要绑定方法
      updateNum2: "updateNum2",
    },
  },
  methods: {
    btnHandler2(e) {
      this.updateNum2(e.target.dataset.step);
    },
  },
});
```

```html
<view> {{numA}}+{{numB}}={{sum}} </view>
<van-button bindtap="btnHandler2" type="primary" data-step="{{1}}">
  numA+1
</van-button>
```

# 分包的使用 每个分包不能超 2M，主包和分包合起来不能超 16M

### 详情中能看到分包的大小

在 app.json 的 subpackages 节点中声明分包的结构

```js
{
  "pages":[//主包的所有页面
    "pages/index",
    "pages/logs"
  ],
  "subpackages":[//通过 subpackages节点，声明分包的结构
    {
      "root":"packageA",//第一个分包的根目录
      "pages":[
        "pages/cat",
        "pages/dog"
      ]
    },{
      "root":"packageB",//第一个分包的根目录
      "name":"pack2",//分包的别名
      "pages":[//当前分包下，所有页面的存放路径都是相对于root，下面的页面都是相对于packageB目录
        "pages/apple",
        "pages/banana"
      ]
    }
  ]
}
```

### 打包原则

1. 小程序会按 subpackages 的配置进行分包，subpackages 之外的的目录将被打包到主包中
2. 主包也可以有自己的 pages（既最外层的 pages 字段）
3. tabBar 页面必须在主包内
4. 分包之间不能互相嵌套

### 引用原则

1. 主包无法引用分包内的私有资源
2. 分包之间不能互相引用私有资源
3. 分包可以引用主包内的公共资源

## 分包---独立分包

1. 什么是独立分包
   独立分包本质上也是分包，只不过它比较特殊，可以独立于主包和其他分包而单独运动

### 独立分包和普通分包的区别

最主要的区别：是否依赖于主包才能运行

1. 普通分包必须依赖于主包才能运行
2. 独立分包可以在不下载主包的情况下，独立运行

### 独立分包的应用场景

开发者可以按需，将某些具有一定独立性的页面配置到独立分包中。

### 独立分包的配置方法

通过 independent 声明独立分包

```js
  {
      "root":"packageB",//第一个分包的根目录
      "name":"pack2",//分包的别名
      "pages":[//当前分包下，所有页面的存放路径都是相对于root，下面的页面都是相对于packageB目录
        "pages/apple",
        "pages/banana"
      ],
      independent:true //通过此节点，声明当前moduleB 分包为“独立分包”
    }
```

### 独立分包的引用用原则

独立分包和普通分包以及主包之间，是相互隔绝的，不能相互引用彼此的资源~ 1.主包无法引用独立分包内的私有资源 2.独立分包之间，不能相互引用私有资源 3.独立分包和普通分包之间，不能相互引用私有资源 4.特别注意：独立分包中不能引用主包内的公共资源

## 分包预下载

在进入小程序的某个页面时，由框架自动与下载可能需要的分包，从而提升进入后续分包页面时的启动速度
预下载分包的行为，会在进入指定的页面时触发。在 app.json 中，使用 preloadRule 节点定义分包的与下载规则

```js
  {
     "preloadRule":{//分包预下载的规则
      "pages/contact/contact":{//触发分包预下载的页面路径
      //network 表示在指定的网络模式下进行预下载，
      //可选值为：all(不限网络)和WiFi（仅WiFi模式下进行预下载）
      //默认值：WiFi
        "network":"all",
        //packages 表示进入页面后，预下载哪些分包
        //可以通过 root 或name 指定预下载哪些分包
        "packages":["pkgA"]
      }
     }
    }
```

### 分包预下载的限制

同一个分包中的页面象有的共同的预下载大小限额 2M,例如
主包:home--A 1M, message--B 1M,contact---C 1M
上面是不允许的，因为主包中的预下载体积超过 2M
