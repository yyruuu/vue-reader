<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
document.addEventListener('DOMContentLoaded',()=>{
  //找到根元素
  const html = document.querySelector('html');
  //当屏幕的宽度很大时，字体也会很大 所以要对字体的大小做一个限制
  let fontSize = window.innerWidth / 10;
  //三元运算符，限制fontSize的值最大为50
  fontSize = fontSize > 50 ? 50 : fontSize;
  //将跟元素的fontSize值设置为屏幕的十分之一
  html.style.fontSize = fontSize + 'px';
})
</script>

<style>

</style>
