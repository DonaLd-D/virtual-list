#### 虚拟长列表

- 需求：在不分页的情况下一次性渲染上万条数据且页面不卡顿
- 已有解决方案：
  - [https://github.com/tangbc/vue-virtual-scroll-list](https://github.com/tangbc/vue-virtual-scroll-list) 
  - [https://github.com/Akryum/vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller)
- 实现思路：对长列表进行切片，只展示看到的部分。滚动时根据滚动的距离切片数组的长度。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>虚拟长列表</title>
  <meta name="viewport" content="width=device-width">
</head>
<body>
  <h1>虚拟长列表</h1>
<div id="app"></div>
  
<script src="https://unpkg.com/vue@next"></script>
<script>
  let t = Date.now()
  const  { ref, reactive, computed, onMounted, createApp } = Vue;
  createApp({
    template: `  
      <div class="container" @scroll="onScroll">
        <div class="panel" ref="panel"
             :style="{paddingTop: paddingTop + 'px'}" >
          <div class="item"  :key="item" v-for="item in visibleList">
            {{ item }}
          </div>
        </div>
      </div>`,
    
    setup() {
      let panel = ref(null)  //列表容器DOM
      let raw = Array(100000).fill(0).map((v, i) => `item-${i}`); //构造的长列表原始数据
      let count = 50;      //实际渲染DOM的列表数量
      let start = ref(0);  //从长列表数组总截取数据的起点 
      let end = ref(50);   //从长列表数组总截取数据的终点
      let itemHeight = 40; //单个列表项的高度
      let itemWidth = 130;  //单个列表项的宽度
      let paddingTop = ref(0); //列表容器的上内边距
      let panelWidth=600*0.98 //列表容器的宽度
      let colNumber=Math.floor(panelWidth/itemWidth)  //一行能够容纳列表项的数量
      let totalHeight = Math.ceil(raw.length/colNumber)*itemHeight  //原始数据理论上完全渲染后占据的总高度
      let visibleList = computed(() => raw.slice(start.value, end.value)); //根据起点和终点获取要渲染的数据
      
      onMounted(() =>{
        panel.value.style.height = totalHeight + 'px'
      }) //在mounted后设置列表容器的高度
      
      //滚动-->根据滚动距离计算起点和终点的下标-->计算属性得到visibleList-->真实DOM被替换 同时设置paddingTop让元素视觉上没跳动
      const onScroll = function (e) {
        start.value = Math.floor(e.target.scrollTop / itemHeight)*colNumber; //当滚动后，重新计算起点的位置
        end.value = start.value + count; //设置终点的位置
        paddingTop.value = start.value*itemHeight/colNumber; 
      };

      return {
        visibleList, paddingTop, panel, onScroll
      };
    }
  }).mount('#app');


</script>

<style>
  * {
    box-sizing: border-box;
    margin: 0;
  }
  .container {
    height: 300px;
    width:600px;
    border:1px solid red;
    overflow-y: scroll;
  }
  .panel{
    border:1px solid green;
    margin:0 5px;
    width:98%;
    float:left;
  }

  .item {
    display:inline-block;
    border: 1px solid blue;
    line-height: 30px;
    height: 30px;
    width:120px;
    padding: 0 10px;
    cursor: pointer;
    margin:5px 5px;
  }

</style>
  
</body>
</html>
```
