# Vue高仿饿了么外卖APP
## 主要技术栈
>- Vue.js
>- vue-cli 脚手架搭建项目
>- vue-router 实现路由切换
>- webpack 打包项目文件
>- ES6 + ESlint
>- flex 弹性布局
>- stylus 编写样式
>- Vue 过渡动画
>- 联动滚动借助了 better-scroll 插件
>- localStorage 本地数据存取

## 实现功能
>- Goods、Ratings、Seller 组件视图均可上下滚动
>- 商品页 点击左侧menu，右侧list对应跳转到相应位置
>- 点击list查看商品详情页，父子组件的通信
>- 评论内容可以筛选查看
>- 购物车组件，包括添加删除商品及动效，购物控件与购物车组件之间为兄弟组件通信，点击购物车图标，展示已选择的商品列表
>- 商家实景图片可以左右滑动
>- loaclStorage 缓存商家信息（id、name）

## Build Setup
```
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```


## 要求自适应的布局
### 1、左侧宽度固定，右侧宽度自适应
```
// 左侧固定width：80px，右侧自适应
parent:
    display:fiexd;
child-left:
    flex:0 0 80px
child-right:
    flex:1
```
### 2、元素宽度自适应设备宽度，且元素要求等宽高样式
```
.img-header 
    position:relative
    width:100% // width是 设备宽度
    height:0
    padding-top:100% // 高度设为0,使用padding撑开
    .img 
        position:absolute //定位布局
        top:0
        left:0
        width:100%
        height:100%
 
```
## seller组件出现的问题
### 1、打开seller页面，无法滚动
分析：出现这种现象是因为better-scroll插件是严格基于DOM的，数据是采用异步传输的，页面刚打开，DOM并没有被渲染，所以，要确保DOM渲染了，才能使用 better-scroll
</br>
解决方案：用到mounted钩子函数，同时必须搭配`this.$nextTick()`

### 2、在seller页面，刷新后，无法滚动
分析：出现这种情况是因为mounted函数在整个生命周期中只会只行一次 
</br>
解决方案：使用watch方法监控数据变化，并执行滚动函数` this._initScroll();this._initPicScroll();`

## sticky-footer布局
在 header 组件的详情页采用 sticky-footer 布局，主要特点是如果页面内容不够长的时候，页脚块粘贴在视窗底部；如果内容足够长时，页脚块会被内容向下推送

实现：
父级 position:fixed，内容设 为padding-bottom:64px，页脚相对定位，margin-top:-64px，clear:both 为了保证兼容性，父级要清除浮动
## 背景模糊效果
`filter：blur(10px)`,注意，所有在内的子元素也会模糊，包括文字，所以采用定位布局，背景单独占用一个层，ios有一个设置backdrop-filter:blur(10px)，只会模糊背景,但不支持android

## goods,ratings,seller组件之间切换时会重新渲染
解决方案：在 app.vue 内使用 keep-alive，保留各组件状态，避免重新渲染
```
<keep-alive>
    <router-view :seller="seller"></router-view>
</keep-alive>
```
## transition过渡
过渡有六种状态
>- v-enter：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的**下一帧**移除。
>- v-enter-active：定义进入过渡生效时的状态。个类可以被用来定义**进入过渡的过程时间，延迟和曲线函数。**
>- v-enter-to：在离开过渡被触发时**立刻**生效，下一帧被移除。
>- v-leave：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，**下一帧**被移除。
>- v-leave-active：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义**离开过渡的过程时间，延迟和曲线函数。**
>- v-leave-to：定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 v-leave 被删除)，在过渡/动画完成之后移除。
```
// 例如
    opacity:1
    background: rgba(7,17,27,0.8)
    .fade-enter-active,.fade-leave-active
      transition:all 0.5s
    /* 若fade-leave-active改成fade-leave，则动画立即消失，因为在触发的下一帧动画就结束了，离开的动画过程就没有显示 */    
    .fade-enter,&.fade-leave-active
      opacity:0
      background: rgba(7,17,27,0)
```

## ref的使用
- $refs的使用是vue操作dom的一种方式:

- ref 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 $refs 对象上。

-  如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素; 如果用在子组件上，引用就指向组件实例:


## 星星组件star.vue
整个流程是:
>- 绑定星星类型的class(48,36,24尺寸),使用starType
>- 使用class来显示星星,有3种类型,全星,半星,无星,使用star-item代表星星本身,然后分别使用on,off,half代表三种不同类型的星星
>- 一个span代表一个星星项目,并且使用v-for循环将星星项目输出
#### html部分
```
<template>
  <div class="star" :class="starType">
    <span v-for="(itemClass,index) in itemClasses" v-bind:key="index" :class="itemClass" class="star-item" ></span>
  </div>
```
#### js部分
- 设置常量是为了方便解耦
- 星星计算比较巧妙(根据分数转换为星星数)
    - 对于分数score进行乘以2然后向下取整，然后再除以2，是为了获取所有星星的数量，并且这个数量是0.5倍数的,例如4.6 2就是9.2，然后向下取整是9，然后再除以2就是4.5，那么就可以得到一个0.5倍数的星星数，可以转换为4个全星+一个半星
    - 对于非整数的星星算作是半个星星，需要知道是否有存在这种情况，所以分数score%1 ，例如 8 % 1是0，8.5 % 1就不是0，并且这个半星只会出现一次，因为半星状态就只要一个
    - 没有星星的部分是要补全的，这里使用while循环来处理这种情况
```
<script>
  //设置常量
  const LENGTH = 5;
  const CLS_ON = 'on';
  const CLS_HALF = 'half';
  const CLS_OFF = 'off';

  export default{
    props: {
      size: { //传入的size变量
        type: Number //设置变量类型
      },
      score: { //传入的score变量
        type: Number
      }
    },
    computed: {
      starType(){ //通过计算属性,返回组装过的类型,用来对应class类型
        return 'star-' + this.size;
      },
      itemClasses(){
        let result = []; //返回的是一个数组,用来遍历输出星星
        let score = Math.floor(this.score * 2) / 2; //计算所有星星的数量
        let hasDecimal = score % 1 !== 0; //非整数星星判断
        let integer = Math.floor(score); //整数星星判断
        for (let i = 0; i < integer; i++) { //整数星星使用on
          result.push(CLS_ON);//一个整数星星就push一个CLS_ON到数组
        }
        if (hasDecimal) { //非整数星星使用half
          result.push(CLS_HALF);//类似
        }
        while (result.length < LENGTH) { //余下的用无星星补全,使用off
          result.push(CLS_OFF);//类似
        }
        return result;
      }
    }
  }
</script>
```
#### CSS部分
有3中不同尺寸的星星，分别为48px、36px、24px，不同尺寸的星星分别使用不同的图片。
```
<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin";

  .star
    font-size:0
    .star-item
      display: inline-block
      background-repeat: no-repeat
    &.star-48
      .star-item
        width: 20px
        height: 20px
        margin-right: 22px
        background-size: 20px 20px
        &:last-child   /* 最后一个外边距为0 */
          margin-right: 0
        &.on
          bg-image('star48_on')
        &.half
          bg-image('star48_half')
        &.off
          bg-image('star48_off')
    &.star-36
      .star-item
        width: 15px
        height: 15px
        margin-right: 6px
        background-size: 15px 15px
        &:last-child
          margin-right: 0
        &.on
          bg-image('star36_on')
        &.half
          bg-image('star36_half')
        &.off
          bg-image('star36_off')
    &.star-24
      .star-item
        width: 10px
        height: 10px
        margin-right: 3px
        background-size: 10px 10px
        &:last-child
          margin-right: 0
        &.on
          bg-image('star24_on')
        &.half
          bg-image('star24_half')
        &.off
          bg-image('star24_off')
```

## 学习参考
>- Vue2.0 官网： https://vuefe.cn/v2/guide/
>- vue-cli：https://github.com/vuejs/vue-cli
>- vue-router 2.0文档：https://router.vuejs.org/zh-cn/
>- webpack官网：https://webpack.js.org/
>- better-scroll 插件使用：https://github.com/ustbhuangyi/better-scroll
>- ESlint 代码风格检查：http://eslint.org/docs/rules/
>- ES6 入门: http://es6.ruanyifeng.com/
>- Sticky footers http://www.w3cplus.com/css3/css-secrets/sticky-footers.html
>- Flex 弹性布局: http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool
>- 设备像素比：http://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/
>- 贝塞尔曲线生成器：http://cubic-bezier.com/#.17,.67,.83,.67
