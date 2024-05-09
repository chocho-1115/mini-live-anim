
### 小程序直播间点赞动画

uniapp下，直播间点赞动画

[参考](https://juejin.cn/post/6859702071906533383)
感谢Rileycai的分享

#### 不足之处
generatePathDataReverse函数需要去补充
小程序canvas是置顶的，如果要吧点赞动画置于点赞按钮下面，需要wx.canvasToTempFilePath转为图片。但处于性能问题，这明显不是个好办法。

#### 使用方法
``` html
<like-anim :like="like" />
```

``` javascript
import likeAnim from '@/components/like-anim.vue'
```

like-anim组件会监听参数like的变化来触发动画，所以只需要在父组件给like一个增量即可。