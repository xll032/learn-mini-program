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
setTimeout(function() {
  // do something
}, 200)

// 计时器 es6
setTimeout(() => {
  // do something
}, 200)

// 简单的运算 es5
function calc(a, b) {
  return ((a * 0.7 + b * 0.3) * 3).toFixed(2)
}

// 简单的运算 es6
let calc = (a, b) => ((a * 0.7 + b * 0.3) * 3).toFixed(2)
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
  onLoad(options) {
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
  getList() {
    wx.request({
      url: '/getStatus', // 获取状态
      method: 'POST',
      success: res => {
        if (res.code === 200) {
          wx.request({
            url: '/getList', // 根据状态，获取列表
            type: 'POST',
            data: {
              status: res.status
            },
            success: res => {
              if (res.code === 200) {
                this.setData({
                  list: res.list
                })
              }
            },
            fail: res => {
              console.log('请求出错')
            }
          })
        }
      },
      fail: res => {
        console.log('请求出错')
      }
    })
  }
})
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
  getStatus() {
    return new Promise((resolve, reject) => {
      wx.request({
        url: '/getStatus',
        method: 'POST',
        success(res) {
          if (res.code === 200) {
            resolve(res)
          } else {
            reject(res)
          }
        },
        fail(res) {
          console.log('请求出错')
        }
      })
    })
  },
  /**
   * @func 获取列表
   */
  getList() {
    this.getStatus().then(res => {
      wx.request({
        url: '/getList',
        method: 'POST',
        success(res) {
          if (res.code === 200) {
            that.setData({
              list: res.list
            })
          }
        },
        fail(res) {
          console.log('请求出错')
        }
      })
    })
  }
})
```

> 上面的代码，看起来还是套了很多层，那是因为实际开发中，我们会对 `wx.request` 再做一层封装，最后的代码可能是这样的：

```javascript
const API = require('../../api/api.js') // 引入封装的方法

Page({
  data: {
    list: {}
  },
  /**
   * @func 获取列表
   */
  getList() {
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

- 封装了 `wx.request` 后，请求错误（`fail`）可以统一处理，不必再写；
- 与此同时，后端的数据格式也统一做了处理，统一抛出的 `errorCode` 等判断也不必再写；
- 封装后的方法本身包含了 `url`，`method`，不必再写，只需要在写好的`api.js`中做配置；
- 封装后的方法本身是一个 `Promise`，最早写到的 `getStatus` 也不必写。

- 2） 组件化  
  使用小程序 `template` 将公用部分抽离出去
- q：何时使用？ a：多出使用到，样式相同，功能类似的部分；
- q：需要注意什么？ a：公共部分精准抽离，使用方便。

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
> 虽然不需要掌握记住每一个组件和 API 如何使用，但有哪些组件，哪些能够实现，哪些无法实现，心里一定要有数。这样不仅能在产品会、技术会时提出难点与风险（如果无法实现，也可以提出相应的替代方案，如果无法实现，至少大家也能做好心理准备），而且开发时间的评估也会更加精准。

### 6. 小练习

> 如果毫无经验，可以尝试做一些小东西，提高自己的理解。数据可以静态写死，也可以全局定义一个 api.js 来获取。

- 1）手机中的计算器，注意特殊输入情况下的判断；
- 2）实现一个简易“微信”，包括：“聊天列表页”、“聊天详情页”、“发现页”、发现页中的“朋友圈页”、“我的”、“设置页”、以及“设置页中的隐私页”；
- 3）互动砸蛋小程序（需要看效果的可以找我要，难度比上面两个高）

### 7. 小贴士（将会持续更新）

#### 1）`<image class="demo" src="{{xxx}}" mode="widthFix"></image>`

这种方法可以保证图片的长宽比，但是从后台获取过来的一瞬间，可能会变形，体验不好。可能是因为刚获取到的时候，解析不到图片原本的长宽。

建议的做法：使用样式固定大小，例如：`.demo { width: 100rpx; height: 120rpx; }`

#### 2）`wx.setStorageSync`

这个方法用于同步存储数据到本地，这个方法并不稳定（有极低的可能会失败），如果在主流程中使用将有可能导致重要数据丢失，甚至程序卡死。  
失败的原因主要是：  
a）微信版本过低，不支持；
b）手机内存访问出错；

建议的做法：主流程和重要数据不使用`wx.setStorageSync`，可以使用`globalData`代替，如果希望长期保留数据也可适当使用（但仅限即使没有存到也无关紧要的数据）。此外，使用`wx.setStorageSync`最好使用`try catch`保证代码不报错。

此外，使用`wx.setStorage`会稳定一些，但是要注意这个方法是异步调用的。

#### 3）`border: 1rpx solid #666`

细边框渲染。由于真机上存在 `dpr` 的影响，会导致`1rpx`不渲染，问题比较明显的机型是：iPhone Plus 和 X。

建议的做法：将外部容器的宽高设为奇数或.5 的 2 倍，例如： `.container { width: 101rpx; height: 101rpx }`，`.container { width: 110rpx; height: 110rpx }`  
常见的`dpr`

|        设备        | dpr |
| :----------------: | :-: |
|    iPhone 及 s     |  2  |
|    iPhone Plus     |  3  |
|      iPhone X      |  3  |
| 安卓从 1 到 4 都有 | 1-4 |

注意点：上面的做法实际上并没有办法完全兼容，还是部分机型会出问题。神奇的是，并不是所有 `1rpx` 都会出问题。所以，需要实测一下。如果出现了这种情况，只能退而求其次，使用 `2rpx`

#### 4） `scroll-view` 组件

a）竖向滚动： `<scroll-view class="demo" scroll-y></scroll-view>`，同时给组件加上一个固定高度即可。`.demo { height: 300rpx; }`  
b）横向滚动：有几个注意点（因为一般情况下是会换行的，所以可能会没有想到），在下面的例子进行说明：  
wxml（没什么特别的）：

```html
<scroll-view class="demo">
  <view class="demo-item">A</view>
  <view class="demo-item">B</view>
  <view class="demo-item">C</view>
  <view class="demo-item">D</view>
</scroll-view>
```

wxss（1. 外部元素设置不换行；2. 内部元素设置成行内元素）：

```css
.demo {
  width: 150rpx;
  white-space: nowrap;
}
.demo-item {
  display: inline-block;
  width: 100rpx;
}
```

c）隐藏滚动条
第一时间可能会想到使用官方`API`上的 `scroll-top` 和 `scroll-left` 来实现，不过很不幸，这样做并没有卵用。下面直接上代码。
wxss：

```css
::-webkit-scrollbar {
  width: 0;
  height: 0;
  color: transparent;
}
```

#### 5）`wx:for` 嵌套 `wx:for`

用 `wx:for-item` 给嵌套循环一个别名用来区分默认的 ``直接贴个例子

```javascript
Page({
  data: {
    list: [
      {
        id: 1,
        title: 'demo',
        sub: ['11111', '22222', '33333']
      },
      {
        id: 1,
        title: 'demo',
        sub: ['11111', '22222', '33333']
      }
    ]
  }
})
```

```html
<view wx:for="{{list}}" wx:key="{{item.id}}">
  <view>{{item.title}}</view>
  <view wx:for="{{list.sub}}" wx:for-item="{{cell}}" wx:key="{{index}}">
    {{cell}}
  </view>
</view>
```

#### 6）`button` 边框完全去除

```html
<button>点我</button>
```

```css
button {
  border: none;
  outline: none;
}
```

理论上讲，这样就没有边框了，但是在微信小程序里，还有一个样式会导致有一个淡淡的边框存在
需要加上：

```css
button::after {
  border: none;
}
```

#### 7）阿拉丁统计与分享参数冲突（阿拉丁会覆盖默认分享参数）

原分享路径为：/pages/newsDetail/newsDetail?id=66666
接入阿拉丁后，由于阿拉丁统计需要一个参数，分享链接会修改，从而变为：/pages/newsDetail/newsDetail?ald_share_src=7m2397806r6t72g4354d4y174  
这就导致了分享点击会报错。

解决方法：

- 在`Page`中配置`onShareAppmessage`方法，`return`指定`path`，就可以同时保留两者的参数。
- 示例代码：

```javascript
/**
 * @func 分享事件
 * @returns {object} 分享的参数
 */
onShareAppMessage() {
  // 这里可以写自定义的事件，例如统计
  return {
    path: `/pages/newsDetail/newsDetail?id=${this.data.articleId}`
  }
}
```

再来看一下分享的链接：/pages/newsDetail/newsDetail?id=66666&ald_share_src=7m2397806r6t72g4354d4y174  
完美。

#### 8）小程序触摸穿透

我们经常会遇到弹出层的需求。在 web 中，我们可以通过动态设置 `body { height: 100%; overflow: hidden; }` 来禁止滚动。在小程序中，则需要动态将 page 设置为该属性，但是动态设置 page 是实现不了的（因为页面中并没有这个元素，而小程序中也没有 DOM）。  
解决方案：
给弹出遮罩层加一个触摸事件`catchtouchmove`即可。

```html
<view catchtouchmove="stopMove"></view>
```

```javascript
/**
 * @func 用于禁止触摸，无需写方法
 */
stopMove () {
    return
}
```

#### 9）小程序同一页面多个分享效果

在小程序中，可能会遇到一个页面里有几个不同的分享逻辑（不同链接、不同文案、图片等）。而实现分享只有一种方式，下面来看看要怎么实现：

```html
<button open-type="share" data-type="1">分享1</button>
<button open-type="share" data-type="2">分享2</button>
```

```javascript
onShareAppMessage (res) {
    let type = res.target.dataset.type // 获取不同button的状态
    return {
        title: `类型${type}`
    }
}
```

#### 10）view中使用文本真机无法准确居中

在小程序中，可能会遇到在真机上文本无法居中的情况，示例代码如下：

```html
<view class="box">
  1
</view>
```

```css
.box {
  display: flex;
  width: 32rpx;
  height: 32rpx;
  justify-content: center;
  align-items: center;
  border: 1rpx solid #e84146;
  border-radius: 50%;
  color: #e84146;
  font-size: 22rpx;
  letter-spacing: 0;
}
```

从代码上看似乎没有任何问题，在模拟器上也是好的。事实上，使用Mac打包预览发布，也都正常。但相同代码在Windows下执行发布就会出现向右偏移的问题。

原因是：Windows系统打包的时候，会把文本内容的换行也塞进去，从而使文本前有一个小空格，导致偏移。

解决方案（css无需处理）：

```html
<view class="box">
  <text>1</text>   
</view>
```

只要养成一个出现文字就用`<text>`包裹起来的好习惯，就可以避免给自己和同事埋下这个坑。
