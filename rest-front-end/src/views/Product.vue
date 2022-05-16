<template>
  <div class="body">
    <div class="header"><h1>Product Page</h1><a href="#">♡</a></div>
    <div class="product-card">
      <div v-for="i in product?.data?.images" :key="i">
        <img :src="'http://localhost:1337' + i?.url" alt="" />
      </div>
    <div class="card-text">
      <p class="title">{{ product?.data?.title }}</p>
      <p>{{ product?.data?.description }}</p>
      <div class="price-and-cart"><p class="price">{{ product?.data?.price }} USD</p> <a href="#"><button class="card-text-button">add to cart</button></a></div>
      <hr>
      <p>Бесплатная экспресс доставка за 2 часа по Алматы и Нур-Султан</p>
      <p>Официальная гарантия: 12 месяцев</p>
      <p><b>Комплект поставки</b></p>
      <p>iPhone, Документация, Кабель-переходник с USB-C на Lightning</p>
      <hr>
    </div>
    </div>
  </div>
</template>

<script>
import { useECommerceStore } from "@/stores/e-commerce";
import { storeToRefs } from "pinia";
import qs from "qs";
import { useRoute, useRouter } from "vue-router";
export default {
  setup() {
    const eCommerceStore = useECommerceStore();
    const route = useRoute();
    eCommerceStore.fetchProduct(+route.params.id)
    const { product } = storeToRefs(eCommerceStore);
   return {
      product,
      eCommerceStore
   }
   },
   name: "Product",
   }
    
</script>

<style scoped>
.body {
  width: 1200px;
  margin: 0 auto;
  background-color: #fff;
}
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 50px;
}
.header a {
  font-size: 24px;
  color: #a18a68;
  text-decoration: none;
}
.header a:hover {
  color: #eba69b;
}
.product-card {
  display: flex;
  padding: 30px 50px;
}
.product-card img {
  width: 400px;
  margin-right: 50px;
}
.price-and-cart {
  display: flex;
  align-items: center;
}
.card-text-button {
  padding: 10px 20px;
  border: #ffffff solid 0px;
  background-color: #eba69b;
  border-radius: 4%;
  color: #fff;
  font-weight: 700;
  margin: 20px 0px;
  opacity: 0.8;
}
.card-text-button:hover {
  opacity: 1;
}
.title {
  font-size: 28px;
  font-weight: 700;
}
.price {
  font-size: 24px;
  font-weight: 700;
  color: #a18a68;
  margin-right: 15px;
}
</style>