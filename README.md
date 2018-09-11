# 小程序学习
> 小程序学习曲线很平稳，总体上手难度很小，可能在开发中会遇到一些坑。  

> 下面是一个入门指南以及一些建议和踩坑提醒。

### 1. 理解 MVVM
- `Model-View-ViewModel`，即将`MVC`的`View`进一步划分，使得视图层和逻辑层区分开来。   
可以简单理解为：`html`和`css`是视图层的部分，`js`是逻辑层的部分。当然`html`中仍保留了一部分逻辑，也是和`js`交互的一个媒介。这种感觉，就好像`html`是“前端”，而`js`则承担了“后端”的角色。  
- 这种解释，不是很准确，却非常形象，可以很好的帮我们理解这个全新的思想。

### 2. MVVM 的特点  
- 1）低耦合。`MVVM`的结构非常清晰，视图层和逻辑层耦合度非常低，无论是代码的阅读，还是可读性，都非常高。即使是一个菜鸟写出来的代码，也比使用传统的`jQuery`干净利落很多（我没有黑`jQuery`的意思，毕竟是一个时代）。这也利于静态页面和代码逻辑开发的解耦，无论是多人协同开发，还是后期添加逻辑，都简单明了许多。  
- 2）代码复用。使用了`MVVM`之后，代码的拆分变得非常容易，从而逻辑复用和组件化简单许多。  
- 3）代码测试。如果视图层没有改变，自动化测试的逻辑基本不会发生变化。而传统的开发，很容易就破坏了原来的结构。  

### 3. 技能要求
- 1）常用的`es6`语法  
a） 箭头函数（要求：了解箭头函数和普通函数的区别，不可乱用）  
例如：  
```javascript
// 计时器 es5
setTimeout(function () {
    // do something
}, 200)

// 计时器 es6
setTimeout(() => {
    // do something
}, 200)

// 简单的运算 es5
function calc (a, b) {
    return ((a * .7 + b * .3) * 3).toFixed(2)
}

// 简单的运算 es6
let calc = (a, b) => ((a * .7 + b * .3) * 3).toFixed(2)
```  
b) 对象中函数使用简写  
不使用：`onLoad: function (options) { }` 这样的写法  
例如：  
```javascript
Page({
    data: {
        // 数据
    },
    /**
     * @func 页面加载 - 生命周期函数
     * @param {object} 传入的参数
     */
    onLoad (options) {
        // 生命周期函数
    }
})
```  
c） Promise  
出现异步操作的时候，使用 `Promise` 来保证程序执行的顺序，同时提高简洁性  
例如：  
```javascript
Page({
    data: {
        list: {} // 列表数据
    },
    /**
     * @func 获取列表数据
     */
    getList () {
        let that = this
        wx.request({
            url: '/getStatus', // 获取状态
            type: 'POST',
            success: function (res) {
                if (res.code === 200) {
                    wx.request({
                        url: '/getList', // 根据状态，获取列表
                        type: 'POST',
                        data: {
                            status: res.status
                        },
                        success: function (res) {
                            if (res.code === 200) {
                                that.setData({
                                    list: res.list
                                })
                            }
                        }
                    })
                }
            }
        })
    }

```  
_划重点：这样写的，拖出去打死。。。_  
> 为什么要打死？  
- 1. 获取列表数据做的事情不仅仅是获取列表，还做了获取状态这件事。 _一个函数只做一件事_  
- 2. 多层嵌套，代码可读性极差，维护成本极高。 _可读性，扩展性_  
建议的写法：
```javascript
Page({
    data: {
        list: {}
    },
    /**
     * @func 获取状态
     */
    getStatus () {
        return new Promise((resolve, reject) => {
            wx.request({
                url: '/getStatus',
                type: 'POST',
                success (res) {
                    if (res.code === 200) {
                        resolve(res)
                    } else {
                        reject(res)
                    }
                }
            })
        })
    },
    /**
     * @func 获取列表
     */
    getList () {
        this.getStatus().then(res => {
            wx.request({
                url: '/getList',
                type: 'POST',
                success (res) {
                    if (res.code === 200) {
                        that.setData({
                            list: res.list
                        })
                    }
                }
            })
        })
    }
})
```  
> 上面的代码，看起来还是套了很多层，那是因为实际开发中，我们会对 `wx.request` 在做一层封装，最后的代码可能是这样的：  
```javascript
const API = require('../../api/api.js') // 引入封装的方法

Page({
    data: {
        list: {}
    },
    /**
     * @func 获取列表
     */
    getList () {
        API.GET_STATUS().then(res => {
            API.GET_LIST().then(res => {
                this.setData({
                    list: res.list
                })
            })
        })
    }
})
```  
d）封装
在上面的方法突然变得无比简洁，也就引出了封装的好处：
- 封装了 `wx.request` 后，请求错误可以统一处理，不必再写；
- 与此同时，后端的数据格式也统一做了处理，统一抛出的 `errorCode` 等判断也不必再写；
- 封装后的方法本身包含了 `url`，`type`，不必再写，只需要在写好的`api.js`中做配置；  
- 封装后的方法本身是一个 `Promise`，最早写到的 `getStatus` 也不必写；  

### 4. 项目结构    
> 这点不多说了，小程序的官方教程写的很清晰明了。传送门：[小程序代码构成](https://developers.weixin.qq.com/miniprogram/dev/quickstart/basic/file.html 'https://developers.weixin.qq.com/miniprogram/dev/quickstart/basic/file.html')

需要注意的是项目的整洁性，建议项目的结构：  
```
|- api  
    |- api.js // 用于封装请求的函数，以及接口  
    |- config.js // 用于域名的配置（测试站、正式站）  
|- images // 存放图片（由于小程序体积最好尽量小，一般图片会放在 oss 上，当然也不排除有些图片需要存在项目中）
    |- 
|- pages  
    |- home // 页面目录，可以使用大驼峰也可以用小驼峰，但要统一  （可以通过新建 Page 一键生成下面的文件）  
        |- home.wxml  
        |- home.wxss  
        |- home.json  
        |- home.js  
|- styles  
    |- storeList.wxss // 可以抽离的公共样式  
|- utils  
    |- md5.js // 工具类函数  
|- app.js // 小程序 app 全局对象  
|- app.json // 小程序公共配置  
|- app.wxss // 小程序全局样式  
|- project.config.json // 配置文件，一般不用管  
```

### 5. 组件及 API
> 建议仔细对照小程序的文档系统学习一下。传送门：[小程序组件](https://developers.weixin.qq.com/miniprogram/dev/component/view.html 'https://developers.weixin.qq.com/miniprogram/dev/component/view.html')  
虽然不需要掌握记住每一个组件和 API 如何使用，但有哪些组件，哪些能够实现，哪些无法实现，心里一定要有数。这样不仅能在产品会、技术会时提出难点与风险（如果无法实现，也可以提出相应的替代方案，如果无法实现，至少大家也能做好心理准备），而且开发时间的评估也会更加精准。

### 6. 小练习  
> 如果毫无经验，可以尝试做一些小东西，提高自己的理解。数据可以静态写死，也可以全局定义一个 api.js 来获取。

- 1）手机中的计算器，注意特殊输入情况下的判断；  
- 2）实现一个简易“微信”，包括：“聊天列表页”、“聊天详情页”、“发现页”、发现页中的“朋友圈页”、“我的”、“设置页”、以及“设置页中的隐私页”；  
- 3）互动砸蛋小程序（需要看效果的可以找我要，难度比上面两个高）

### 7. 踩坑（将会持续更新）
- 1）`<image class="demo" src="{{xxx}}" mode="widthFix"></image>`   
这种方法可以保证图片的长宽比，但是从后台获取过来的一瞬间，可能会变形，体验不好。可能是因为刚获取到的时候，解析不到图片原本的长宽。

建议的做法：使用样式固定大小，例如：`.demo { width: 100rpx; height: 120rpx; }`

- 2）`wx.setStorageSync`  
这个方法用于同步存储数据到本地，这个方法并不稳定（有极低的可能会失败），如果在主流程中使用将有可能导致重要数据丢失，甚至程序卡死。  
失败的原因主要是：  
a）微信版本过低，不支持；
b）手机内存访问出错；

建议的做法：主流程和重要数据不使用`wx.setStorageSync`，可以使用`globalData`代替，如果希望长期保留数据也可适当使用（但仅限即使没有存到也无关紧要的数据）。此外，使用`wx.setStorageSync`最好使用`try catch`保证代码不报错。

此外，使用`wx.setStorage`会稳定一些，但是要注意这个方法是异步调用的。

- 3）`border: 1rpx solid #666`  
细边框渲染。由于真机上存在 `dpr` 的影响，会导致`1rpx`不渲染，问题比较明显的机型是：iPhone Plus 和 X。

建议的做法：将外部容器的宽高设为奇数或.5的2倍，例如： `.container { width: 101rpx; height: 101rpx }`，`.container { width: 110rpx; height: 110rpx }`  
常见的`dpr`  

| 设备 | dpr |  
| :----: | :----: |  
| iPhone及s | 2 |
| iPhone Plus | 3 | 
| iPhone X | 3 |
| 安卓从1到4都有 | 1-4 |


注意点：上面的做法实际上并没有办法完全兼容，还是部分机型会出问题。神奇的是，并不是所有 `1rpx` 都会出问题。所以，需要实测一下。如果出现了这种情况，只能退而求其次，使用 `2rpx`


