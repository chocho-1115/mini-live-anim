
### 小程序直播间点赞动画

uniapp下，直播间点赞动画

[参考](https://juejin.cn/post/6859702071906533383)
感谢Rileycai的分享

#### 不足之处
generatePathDataReverse函数需要去补充。

小程序canvas是置顶的，如果要吧点赞动画置于点赞按钮下面，需要wx.canvasToTempFilePath转为图片。但处于性能问题，这明显不是个好办法。

暂无任意处双击点赞功能。

#### 使用方法
``` html
<view class="like-box">
    <like-anim :like="like" />
</view>
<style lang="scss" scoped>
.like-box{
    position:absolute;
    pointer-events: none;
    /* 定位在点赞按钮的位置 bottom:90%;left:50%;transform: translateX(-50%); */
}
</style>
```

``` javascript
import likeAnim from '@/components/like-anim.vue'
```

like-anim组件会监听参数like的变化来触发动画，所以只需要在父组件给like一个增量即可。
因小程序的代码体积限制，静态图片建议移至oss上。