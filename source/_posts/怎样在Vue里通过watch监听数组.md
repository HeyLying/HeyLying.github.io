---
title: 怎样在Vue里通过watch监听数组
date: 2020-06-14 16:21:50
categories: 前端框架
---

在业务代码里，组里发现Vue的watch在使用中的一个细节问题。

场景：三维模型的渲染需要一组坐标数据信息（包括经纬高），假设我们现在需要更新数组其中一个坐标，并且在position发生变化时，通过监听，实时刷新。

<!-- more -->



> 当bad_compute_method = true：

因为 `r.push({ x: coord.lat * 10, y: coord.lng * 10, z: coord.alt * 10 })` 创建一堆新对象放入数组，所以所有的item的watch都被激活了。

（1）点击：good_edit()

```powershell
edited
item_positions recomputed
Item 0 watched
Item 1 watched
Item 2 watched
```

（2）点击：bad_edit()，与good_edit()表现一致。

```powershell
changed
item_positions recomputed
Item 0 watched
Item 1 watched
Item 2 watched
```



> 当bad_compute_method = false：

因为Bad Edit创建了一个新的点坐标对象替换了coordinates里面第一个元素，computed被激活。

（1）点击：good_edit()

```powershell
edited
Item 0 watched
```

（2）点击：bad_edit()

```powershell
changed
item_positions recomputed
Item 0 watched
```



App.vue

```vue
<template>
  <div id="app">
    <button @click="toggle_compute_method">
      Toggle compute method: {{ bad_compute_method ? 'Bad' : 'Good' }}
    </button>
    <button @click="good_edit">Good Edit</button>
    <button @click="bad_edit">Bad Edit</button>
    <div v-for="(position, index) in item_positions" :key="index">
      <Item :index="index" :position="position" />
    </div>
  </div>
</template>

<script>
import Item from './components/Item.vue'
export default {
  name: 'App',
  components: { Item },
  data() {
    return {
      bad_compute_method: false,
      coordinates: [
        { lat: 1, lng: 2, alt: 3 },
        { lat: 4, lng: 5, alt: 6 },
        { lat: 7, lng: 8, alt: 9 },
      ],
    }
  },
  computed: {
    item_positions() {
      console.log('item_positions recomputed')
      const r = []
      for (let idx in this.coordinates) {
        if (idx % 2 !== 0) continue // 假设我们需要挑出第偶数个坐标进行渲染
        const coord = this.coordinates[idx]
        if (this.bad_compute_method) {
          // 不好，有两点问题
          // 1)每次创建的都是一个新的Object，会导致Vue认为item_position全部换新，即使仅有部分值与上次不一样
          // 2)根据Vue的依赖收集原理，compute函数收集到了对coord具体成员lat lng alt的依赖，
          //   会导致每次具体成员被更改时，compute会被执行，整个列表重新计算
          r.push({ lat: coord.lat * 10, lng: coord.lng * 10, alt: coord.alt * 10 })
        } else {
          // 好，computed函数只做高层级映射，即只负责把对应元素组合成item_positions
          // 至于里面是啥，compute函数不需要也不应该知道，自然依赖搜集过程也不会触碰coord的具体成员
          r.push(coord)
        }
      }
      return r
    },
  },
  methods: {
    toggle_compute_method: function () {
      this.bad_compute_method = !this.bad_compute_method
    },
    good_edit: function () {
      console.log('edited')
      // 好的编辑方式，只编辑具体成员，不去整体替换
      this.coordinates[0].lat = Math.random()
      this.coordinates[0].lng = Math.random()
      this.coordinates[0].alt = Math.random()
    },
    bad_edit: function () {
      console.log('changed')
      // 不好的编辑方式，整体替换了个新Object，激活了一次item_positions的compute
      this.coordinates.splice(0, 1, {
        lat: Math.random(),
        lng: Math.random(),
        alt: Math.random(),
      })
    },
  },
}
</script>
```



Item.vue

```vue
<template>
  <div>
    <div>{{ render_position.lat }}</div>
    <div>{{ render_position.lng }}</div>
    <div>{{ render_position.alt }}</div>
    <div>---------</div>
  </div>
</template>

<script>
export default {
  name: 'Item',
  props: ['position', 'index'],
  data() {
    return { render_position: {} }
  },
  watch: {
    position: {
      handler(new_pos) {
        // 这里我们故意watch这个position成员，把Item假象成需要三维渲染的物件，
        // 坐标变化后需要手动watch并更新render_position
        console.log(`Item ${this.index} watched`)

        this.render_position = {
          lat: new_pos.lat,
          lng: new_pos.lng,
          alt: new_pos.alt,
        }
      },
      immediate: true,
      deep: true,
    },
  },
}
</script>
```



通过上述例子可以发现：

对象数组中，不必总是需要通过splice创建新的对象。每个对象中的元素可以直接进行更改并且能够触发视图更新，但是如果需要通过watch来监听这个数组是否发生变化，必须加上`deep:true`。