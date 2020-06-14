---
title: 怎样在Vue里通过watch监听数组
date: 2020-06-14 16:21:50
tags: 前端框架
---

在业务代码里，发现Vue的watch在使用中的一个细节问题。

场景：三维模型的渲染需要一组坐标数据信息（包括经纬高），假设我们现在需要更新数组其中一个坐标，并且在position发生变化时，通过监听，实时刷新。

```javascript
data () {
    return {
        positionList: [
            {lat: 10, lng: 20, alt: 30},
            {lat: 100, lng: 200, alt: 300},
            {lat: 1000, lng: 2000, alt: 3000}
        ]
    }
},
methods: {
    handleClick () {
        // 如果是对象数组，可以通过这种方法改变元素的值，并且能够触发视图更新
        this.positionList[0].alt = item.alt / 10
    }
},
watch: {
    // dataList (newVal) { // 无法监听到数组变化
    //     newVal.forEach(item => {
    //        console.log(item.alt)
    //     })
    //  },
    positionList: { // 通过设置deep的值可以监听到
        handler (newVal) {
            newVal.forEach(item => {
                console.log(item.alt)
            })
        },
        deep: true
    }
}
```





