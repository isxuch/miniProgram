# 微信小程序相关
> addGwc (加入购物车效果) 使用说明
1. 在app.js中全局定义屏幕宽高/初始化宽高/定义赛贝尔曲线方法
```
//app.js
App({
    onLaunch: function () {
        let _this_ = this;
        that.screenSize();
    },
    //获取屏幕[宽、高]
    screenSize: function () {
        var _this_ = this;
        wx.getSystemInfo({
        success: function (res) {
            _this_.globalData.ww = res.windowWidth;
            _this_.globalData.hh = res.windowHeight;
        }
        })
    },
    // 赛贝尔曲线
    /**
    * @param sx 起始点x坐标
    * @param sy 起始点y坐标
    * @param cx 控制点x坐标
    * @param cy 控制点y坐标
    * @param ex 结束点x坐标
    * @param ey 结束点y坐标
    * @param part 将起始点到控制点的线段分成的份数，数值越高，计算出的曲线越精确
    * @return 贝塞尔曲线坐标
    */
    bezier: function (points, part) {
        let sx = points[0]['x'];
        let sy = points[0]['y'];
        let cx = points[1]['x'];
        let cy = points[1]['y'];
        let ex = points[2]['x'];
        let ey = points[2]['y'];
        var bezier_points = [];
        // 起始点到控制点的x和y每次的增量
        var changeX1 = (cx - sx) / part;
        var changeY1 = (cy - sy) / part;
        // 控制点到结束点的x和y每次的增量
        var changeX2 = (ex - cx) / part;
        var changeY2 = (ey - cy) / part;
        //循环计算
        for (var i = 0; i <= part; i++) {
        // 计算两个动点的坐标
        var qx1 = sx + changeX1 * i;
        var qy1 = sy + changeY1 * i;
        var qx2 = cx + changeX2 * i;
        var qy2 = cy + changeY2 * i;
        // 计算得到此时的一个贝塞尔曲线上的点
        var lastX = qx1 + (qx2 - qx1) * i / part;
        var lastY = qy1 + (qy2 - qy1) * i / part;
        // 保存点坐标
        var point = {};
        point['x'] = lastX;
        point['y'] = lastY;
        bezier_points.push(point);
        }
        return {
        'bezier_points': bezier_points
        };
    },
    globalData: {
        // 屏幕宽度
        ww: '',
        // 屏幕高度
        hh: ''
    }
}}
```
2. 在*.wxml页面中定义需要过度的点
```
<!--index.wxml-->
<view class="container">
  <view class="add-btn" bindtap="addGoods" >
    添加按钮
  </view>
  <view class="good_box" hidden="{{hide_good_box}}" style="left: {{bus_x}}px; top: {{bus_y}}px;">
  </view>
  <view class="gwc">购物车</view>
</view>
```

3. *.wxss定义过度样式
```
page{
  width: 100%;
  height: 100%
}
.good_box {
  width: 40rpx;
  height: 40rpx;
  position: fixed;
  border-radius: 50%;
  overflow: hidden;
  right: 10px;
  top: 50%;
  z-index: 99;
  background: rgb(255, 102, 0);
}

.add-btn{
  width: 100px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  background: greenyellow;
  margin: 40rpx auto;
}

.gwc{
  width: 100%;
  height: 50px;
  text-align: center;
  line-height: 50px;
  background: #fff;
  color: #000;
  position: fixed;
  bottom: 0;
  border-top: 1px solid #bfbfbf;
}
```

4. *.js

```
const app = getApp()
Page({
    data:{
        hide_good_box: true,
    },
    onLoad: function () {
        this.busPos = {};
        this.busPos['x'] = app.globalData.ww/2; //1.4修改轨迹结束时x轴的位置，2是在正中心
        this.busPos['y'] = app.globalData.hh - 10;
    },
    // 加入购物车
  addGoods: function (e) {
    // const query = wx.createSelectorQuery()
    // query.select('#goodsNum').boundingClientRect()
    // query.selectViewport().scrollOffset()
    // query.exec(function (res) {
    //   res[0].top // #the-id节点的上边界坐标
    //   res[1].scrollTop // 显示区域的竖直滚动位置
    //   console.log(res[0].top, '8888')
    // })

    // 如果good_box正在运动
    if (!this.data.hide_good_box) return;
    this.finger = {};
    var topPoint = {};
    this.finger['x'] = e.touches["0"].clientX;
    this.finger['y'] = e.touches["0"].clientY;
    if (this.finger['y'] < this.busPos['y']) {
      topPoint['y'] = this.finger['y'] - 150;
    } else {
      topPoint['y'] = this.busPos['y'] - 150;
    }
    topPoint['x'] = Math.abs(this.finger['x'] - this.busPos['x']) / 2;
    if (this.finger['x'] > this.busPos['x']) {
      topPoint['x'] = (this.finger['x'] - this.busPos['x']) / 2 + this.busPos['x'];
    } else {
      topPoint['x'] = (this.busPos['x'] - this.finger['x']) / 2 + this.finger['x'];
    }
    this.linePos = app.bezier([this.finger, topPoint, this.busPos], 20);
    this.startAnimation();
  },
  startAnimation: function () {
    var index = 0,
      that = this,
      bezier_points = that.linePos['bezier_points'],
      len = bezier_points.length - 1;
    this.setData({
      hide_good_box: false,
      bus_x: that.finger['x'],
      bus_y: that.finger['y']
    })
    this.timer = setInterval(function () {
      index++;
      that.setData({
        bus_x: bezier_points[index]['x'],
        bus_y: bezier_points[index]['y']
      })
      if (index >= len) {
        clearInterval(that.timer);
        that.setData({
          hide_good_box: true,
        })
      }
    }, 15);
  },
})
```


