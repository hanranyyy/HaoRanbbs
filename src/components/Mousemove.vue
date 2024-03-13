<!-- 图片跟随鼠标移动 -->
<template>
  <div class="container">
    <img src="../assets/sheep.png" class="pic" />
  </div>
</template>
  
<script setup>
import { onMounted } from "vue";

// 在组件挂载后执行回调函数
onMounted(() => {
  const container = document.querySelector(".container");
  const pic = document.querySelector(".pic");
  // 监听鼠标移动事件,当鼠标移动时，触发回调函数。
  document.addEventListener("mousemove", function (e) {
    const x = e.clientX; /*返回鼠标相对于浏览器窗口可视区的X坐标，确保页面向下滚动是效果不会消失*/
    const y = e.clientY;
    pic.style.left = x - 4 + "px";
    pic.style.top = y - 55 + "px";
  });
});
</script>
  
<style lang="scss" scoped>
.container {
  position: fixed; /* 使用 fixed 定位确保组件覆盖在页面最上层 */
  z-index: 9999; /* 设置较高的 z-index 值 */
}

.pic {
  position: absolute;
  width: 50px;
  height: 50px;
}
</style>
  