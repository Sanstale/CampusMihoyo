
> 截止至笔者书写时间：2025年10月17日1：36AM，已知的，已上线的业务逻辑如下： 

``` javascript

   judgeIn: function(t, e) {
       return t > 31.19141 && t < 31.193705 && e > 121.594352 && e < 121.596808 || t > 31.052121 && t < 31.053421 && e > 121.752672 && e < 121.753916 || t >
     31.221011 && t < 31.222512 && e > 121.630334 && e < 121.632343 || t > 31.318217 && t < 31.31997 && e > 121.392548 && e < 121.393845 || t > 31.318391 && t <
     31.320292 && e > 121.396041 && e < 121.39726 || t > 31.275604 && t < 31.277297 && e > 121.456016 && e < 121.457606 || t > 31.376768 && t < 31.378306 && e >
     121.248733 && e < 121.250344
   }
```
> 该函数定义了多个坐标，如果用户当前的坐标（纬度 t 和经度 e）在这些区域中的任何一个之内，函数就会返回 true，否则返回 false。

``` javascript
  winthInRange: function(t) {
    var e = this;
    wx.getLocation({
      type: "gcj02",
      isHighAccuracy: !0,
      highAccuracyExpireTime: 5e3,
      success: function(n) {
        var a = n.latitude,
          i = n.longitude;
        e.judgeIn(a, i) ? e.getInfoOrStart(t, !1) : wx.showModal({
          title: "提示",
          content: "您当前位置不在跑步范围内,请在跑步范围内开始跑步；",
          showCancel: !1
        })
      },
      fail: function(t) {}
    })
  }
```
> 上述为在指定区域内开始跑步

``` javascript

  commonRecord: function(e, i) {
    var o = this,
      s = this.data,
      c = s.fitnessId,
      l = s.gradeType,
      r = s.strollRecordId,
      u = s.strollDistance,
      d = o.data.polyline[0].points,
      f = [],
      m = 0;
    d.map((function(e) {
      o.judgeIn(e.latitude, e.longitude) || (m += 1), f = [].concat(t(f), [{
        y: e.latitude,
        x: e.longitude
      }])
    }));
    var g = {
        map: f
      },
      h = {
        fitnessId: c,
        gradeType: l,
        strollDistance: u,
        submitTimestamp: Date.parse(new Date),
        strollRecordId: r,
        strollDetail: JSON.stringify(g)
      };
    e ? (i && (h.strollDistance = 0), m > d.length / 2 ? wx.showModal({
      title: "提示",
      content: "本次成绩无效，系统检测跑步数据异常，需进一步核实，如有疑问请及时联系管理员",
      confirmText: "确定结束",
      cancelText: "继续跑步",
      success: function(t) {
        t.confirm && (h.strollDistance = 0, o.setEndRun(h))
      }
    }) : o.setEndRun(h)) : (a.default.post(n.default.saveStroll, h), o.setRecordPath())
  },
  _startMethods: function(t) {
    var n = this;
    n.getPosChange(), n.data.timer = setInterval((function() {
      var t = n.data.startTemp,
        a = (Date.parse(new Date) - t) / 1e3,
        i = e.default.calcTimes(a);
      n.setData({
        timeNum: a,
        time: i
      })
    }), 1e3), n.setData({
      status: t
    }), n.setRecordPath()
  },

```

> 上述逻辑说明：1/2以上的跑步轨迹需在指定区域内: commonRecord 函数会遍历所有的路径点，计算出在围栏外的点的数量 m，如果 m 超过总路径点数量 d.length 的一半，则成绩无效。

``` javascript

  endclick: function() {
    var t = this;
    wx.getLocation({
      type: "gcj02",
      isHighAccuracy: !0,
      highAccuracyExpireTime: 5e3,
      success: function(e) {
        var n = e.latitude,
          a = e.longitude;
        t.judgeIn(n, a) ? t.commonRecord(!0, !1) : wx.showModal({
          title: "提示",
          content: "您当前位置不在跑步范围内,请在跑步范围内结束；",
          confirmText: "仍然结束",
          cancelText: "重新定位",
          success: function(e) {
            e.confirm ? t.stillEnd() : e.cancel && t._initGetPos(!1)
          }
        })
      },
      fail: function(t) {}
    })
  },

```
> 在指定区域内结束跑步: endclick 函数在结束跑步时，会先判断当前位置。如果不在围栏内，则会警告用户，若用户执意结束，则调用 stillEnd 函数，最终将跑步距离判定为不合格（距离清零）。


## 小结

三个守则：

* 在指定区域内开始跑步
* 一半以上跑步轨迹需在指定区域内
* 在指定区域内结束跑步

我觉得已经很明确了，想要解决校园跑问题，最简单的方式就是虚拟定位，其次有大佬的话可以考虑一下把逆向的程序跑起来然后中间人攻击一下或者其他操作。

也请鼠鼠在发现小程序业务逻辑更新时揣一下窝，毕竟现在小程序逆向相当容易，有意见想法在issue随便提就好
