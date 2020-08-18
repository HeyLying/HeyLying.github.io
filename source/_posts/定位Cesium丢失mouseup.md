---
title: 定位Cesium丢失mouseup
date: 2020-08-18 20:46:39
categories: 前端框架
---



已知：项目组里的产品采用 Electron + Vue 的前端技术栈。

最近发现一个奇怪的现象：在新功能页面中，当按下鼠标移动三维模型，若按住不放，把光标拖至 canvas 外释放鼠标，再将鼠标移回 canvas 内，此时即使不再次按下鼠标，也能移动模型，仿佛鼠标被粘连住一样。但在旧功能页面表现正常。

这部分 CesiumJS 代码是几乎沿用旧功能的，那怎么会出现不同表现形式呢？于是开展定位排查。、

<!-- more -->

<br/>

**思路一：是不是被其他业务逻辑影响了？**

这点是首先想到的可能原因。但删除几乎所有其他代码后，问题仍存在。猜想错误。

<br/>

**思路二：阅读源码的鼠标相关操作逻辑，找出此时是什么逻辑缺了？**

结果发现当鼠标移出目标元素 canvas 后，handlePointerUp() 不再被执行，也就是这时没有监听到 mouseup 。我也写了简单的 demo 进行验证，这其实才是正常的。但是为什么旧功能这时能监听到 mouseup 呢？

```javascript
//  cesium.js
function supportsPointerEvents() {
        if (!defined(hasPointerEvents)) {
            //While navigator.pointerEnabled is deprecated in the W3C specification
            //we still need to use it if it exists in order to support browsers
            //that rely on it, such as the Windows WebBrowser control which defines
            //PointerEvent but sets navigator.pointerEnabled to false.
            //Firefox disabled because of https://github.com/AnalyticalGraphicsInc/cesium/issues/6372
            hasPointerEvents = !isFirefox() && typeof PointerEvent !== 'undefined' && (!defined(theNavigator.pointerEnabled) || theNavigator.pointerEnabled);
        }
        return hasPointerEvents;
}
 
function registerListener(screenSpaceEventHandler, domType, element, callback) {
        function listener(e) {
            callback(screenSpaceEventHandler, e);
        }
        element.addEventListener(domType, listener, false);
        screenSpaceEventHandler._removalFunctions.push(function() {
            element.removeEventListener(domType, listener, false);
        });
 }
 
function registerListeners(screenSpaceEventHandler) {
        var element = screenSpaceEventHandler._element;
 
        // some listeners may be registered on the document, so we still get events even after
        // leaving the bounds of element.
        // this is affected by the existence of an undocumented disableRootEvents property on element.
        var alternateElement = !defined(element.disableRootEvents) ? document : element;
         
        if (FeatureDetection.supportsPointerEvents()) {     // 返回 true
            registerListener(screenSpaceEventHandler, 'pointerdown', element, handlePointerDown);
            registerListener(screenSpaceEventHandler, 'pointerup', element, handlePointerUp);
            registerListener(screenSpaceEventHandler, 'pointermove', element, handlePointerMove);
            registerListener(screenSpaceEventHandler, 'pointercancel', element, handlePointerUp);
        } else {
            // ...
        }
        registerListener(screenSpaceEventHandler, 'dblclick', element, handleDblClick);
        // ...
}
 
// pointerdown
function handlePointerDown(screenSpaceEventHandler, event) {
        event.target.setPointerCapture(event.pointerId);    // 重点是这句
 
        if (event.pointerType === 'touch') {
            var positions = screenSpaceEventHandler._positions;
            var identifier = event.pointerId;
            positions.set(identifier, getPosition(screenSpaceEventHandler, event, new Cartesian2()));
            fireTouchEvents(screenSpaceEventHandler, event);
            var previousPositions = screenSpaceEventHandler._previousPositions;
            previousPositions.set(identifier, Cartesian2.clone(positions.get(identifier)));
        } else {
            handleMouseDown(screenSpaceEventHandler, event);
        }
}
 
 
// pointerup 进不来
function handlePointerUp(screenSpaceEventHandler, event) {
        if (event.pointerType === 'touch') {
            var positions = screenSpaceEventHandler._positions;
 
            var identifier = event.pointerId;
            positions.remove(identifier);
 
            fireTouchEvents(screenSpaceEventHandler, event);
 
            var previousPositions = screenSpaceEventHandler._previousPositions;
            previousPositions.remove(identifier);
        } else {
            handleMouseUp(screenSpaceEventHandler, event);
        }
}
```

尝试网上搜索了很多资料，比如 electron + mouseup，cesium + mouseup 等等，也看到一些网友有相同的问题。但仍旧没替我回答：为什么旧功能表现正常？

https://blog.csdn.net/zheng12tian/article/details/83877391

https://blog.csdn.net/isea533/article/details/71703442



其实，官方 demo 也有一样的问题：

https://sandcastle.cesium.com/#c=tVbLbhs3FP0VQpuMgCkl+ZGkimzUkd04QNy4kZouqsKgOVcawhxyQHJku4E+IEiXXfUDuuyuTR/5HKdoV/2F3iFnrIddoIGr2Yi8r3Pu4UtTZshUwDkYskMUnJM+WFFk9KW3RaMG9/O+Vo4JBWbUiMmrkSJEqLF+rC+6ZMykhbg0WZDAndDqqUoEZ06bJW/KEn1uu8SZorboQiZ7SmTMwbV91nw0UiM1LpSvRbgBdB/pBGRUGBmTFMQkdc1AI3CnoJxwAiw1kOkp7EkZhTIYgR3m2gpfbKfur8+MwxFTm3RsdLYPEwNgozKBkI86G5u0/WBr637n4ziYtrZoe7u9+aB9vzIEFuW4BAowKbBEqMkc5Yi5lDr9As1M2aizuT0PzoXjKYa2ry1GS7lkSPOVVTkMAMdl6guMjirEOFSLfYk5hDYChWHLnQ8NUhlrk1marpT7vECljcLwSohat7rn3NQN1xBe+EusvroQLEmiVyFNsQyXt1y75ard1foLfLuLk8qdlXugS6qqBCuKxbIYIHAvFdmxuAA5EN8gaGfj4dzLLkrvgDOJno02fpVv5n9nlXChEWcYP4PkoO4vNIoRs7J5r25ecrPo+6rMq2g5uHBdMmrsCcMNGzs8MFVzKpwPPBP11o6a82YWt3ltI1iH0taAZbmEfeZYy0tgW2EpEaIaneCQTuTpNVj5bWODtF3PQ3PXvca3UH5idKES8hJSwSWshXiAqBCWZzf4fwj1Q+0IakAeMym1VmsUvUJYnt3g3vlQ7Y+EPCNDU/CzNXIvQTzG6vxO2g/OhFKQkH7K8Mw4/0asrQNWK3+CwzvR3keymvR1lhuwFvl7Imvh7qHmSLeo78b/tY+v/eU7YCrhzDo8NnjRDrWWp8wcgSqi6lYKd7R/RDBUrj7vA+QPapAzDgdTvNoOQ1BUXX6WgwLKGRbwlaoa1IJ7qvLC7XlBolqZiEvBz5qVOBzhNRKTehLde//2h7+++/GPb19f/fLm6rc3V7///Pe77+/FPoHW13/zUVBPzeJ/4Te8zIE+O/h0eLL//MvPQnN3JfX+3a9//vT2fyH1xXFJqRE3etZdStitl+8TkeXauPKRinBnOMCdgdvGtk5x2cFRbm29wr3WYmovEVMikp1b/n8RLpm16BkX0r9yo8Zur4XxN1Kl9s/78ykYyS7LsLSz+ywYKaW9Fk5vz3RhQ61U/gc

又花了一些时间排查。

原来，答案就在 handlePointDown() 里的这句`event.target.setPointerCapture(event.pointerId)`。**setPointerCapture** 可以确保一个元素持续地接收到一个 pointer 事件，即使这个事件的触发点已经移出了这个元素。自己平时确实很少使用这个 API。

参考：https://developer.mozilla.org/zh-CN/docs/Web/API/Element/setPointerCapture

<br/>

**思路三：那为什么新功能页面也加了这句，就不生效呢？**

经过测试，与 iFrame 相关。这说的通，因为官方 demo 里的 cesium 也是 iFrame 的形式嵌入。

新功能是以 iFrame 的形式嵌入的。当子工程与父工程都不是简单的静态资源 html 页面形式，而是需要部署（涉及 webpack 打包等）时，setPointerCapture 在 chrome 不生效。

没错，经实践发现，在 firefox 里，自己写的页面与官方 demo 都没有这个问题，mouseup 能被监听到，是正常的……

<br/>

结论：该问题与浏览器的内部实现有一定关系。