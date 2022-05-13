<template>
  <div class="cont">
    <RouterLink :to="{name: 'home'}">Home</RouterLink> |
    <RouterLink :to="{name: 'products'}">About</RouterLink>
    <RouterView></RouterView>
 </div>
</template>

<script lang="ts">
export default {
  name: "App",
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans&display=swap');

div {
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
}
.cont {
  width: 1200px;
  margin: 0 auto;
}
a {
  text-decoration: none;
  color: #000;
}
</style>
