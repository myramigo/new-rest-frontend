<template>
  <div class="body">
    <div class="header">
      <h1>shop-OP</h1>
      <div class="navigation">
        <div class="categories" v-for="c in categories.data" :key="c.id">
          <ul>
            <li @click="pushQueryToUrl({categoryId: c.id})">{{ c.title }}</li>
          </ul>
        </div>
      <input type="search" placeholder="search" v-model="searchString" @input="pushQueryToUrl({search: searchString})" />
      </div>
    </div>

    <div class="image-box">
      <div class="image-box-text"><p>The most obvious example of a call to action is a "Buy Now" button, or a "Shop Now" button on your storefront.</p>
      <a href="#"><button class="image-box-button">BUY NOW</button></a></div>
    </div>

    <div class="sort">
    <h1>products</h1>
    <select name="sort" v-model="sort" @change="asd($event)">
      <option value="updatedAt:desc">newest</option>
      <option value="price:asc">price: ascending</option>
      <option value="price:desc">price: descending</option>
    </select>
    </div>
    <div class="products-and-filters">
      <div class="filters">
      <h1>filters</h1>
      <p><input type="checkbox" value='lidar:true' @change="pushQueryToUrl({ spec: $event.target.value })"><label>lidar</label></p>
      <p><input type="checkbox" value='dualsim:true' @change="pushQueryToUrl({ spec: $event.target.value })"><label>dualsim</label></p>
      <p><input type="checkbox" value='touchbar:true' @change="pushQueryToUrl({ spec: $event.target.value })"><label>touchbar</label></p>
    </div>
    <div class="products">
      <div class="product-card" v-for="p in products.data" :key="p.id">
        <div class="product-img">
          <div v-for="i in p.images" :key="i.id">
            <img :src="'http://localhost:1337' + i?.url" alt="">
          </div>
        </div>
        <p>{{ p.title }}</p>
        <p class="price">{{ p.price }} USD</p>
      </div>
      </div>
    </div>
    <div class="pages">
    <div v-for="i in products?.meta?.pagination?.pageCount" :key="i">
      <button @click="pushQueryToUrl({page: i})">{{ i }}</button>
    </div>
    </div>
    <div class="footer">
      <p>find us on <a href="#">instagram</a></p>
    </div>
  </div>
</template>

<script>
import { useECommerceStore } from "@/stores/e-commerce";
import { useRoute, useRouter } from "vue-router";
import { ref, onMounted, watchEffect } from "vue";
import { storeToRefs } from 'pinia'
import qs from 'qs'
export default {
  setup() {
    const eCommerceStore = useECommerceStore();
    const { products } = storeToRefs(eCommerceStore);
    const { categories } = storeToRefs(eCommerceStore);
    const route = useRoute();
    const router = useRouter();
    const sort = ref(route.query.sort || "updatedAt:desc");
    const searchString = ref('')
    const spec = ref()
    let ezQuery = {};
    function asd(event) {
      pushQueryToUrl({ sort: event.target.value });
    }
    function pushQueryToUrl(queryParam) {
      Object.entries(queryParam).forEach(([key, value]) => {

        if (value) {
          if (key == "spec") {
            if (ezQuery[key] == undefined) {
              ezQuery[key] = value
              console.log(1)
              console.log('ezQK',ezQuery[key])
                  console.log('eqq1', ezQuery)
            } else {  
              console.log(11123, ezQuery[key])
              // console.log('lol',ezQuery[key].map(s => s.join(':')+ ',' + value))
              // ezQuery[key] = ezQuery[key]+ ',' + value;
              ezQuery[key] = ezQuery[key].map(s => s.join(':')) + ',' + value;
              
              // ezQuery[key] = queryParam.spec
              console.log(2)
              console.log( 'value',value)
              console.log('ezQuery', ezQuery)
              console.log('ezQK2',ezQuery[key])
            }
          } 
          else {
            ezQuery[key] = value;
          }
        }
        else {
          ezQuery[key] = undefined;
        }
      });
      console.log('qparam', queryParam)

      router.push({ query: ezQuery })
    }
    async function createProductQuery() {
      console.log('nado',ezQuery.spec)
      ezQuery = {
        page: route.query.page,
        gte: route.query.gte,
        lte: route.query.lte,
        sort: route.query.sort,
        categoryId: route.query.categoryId,
        spec: route.query.spec ? route.query.spec.split(',').map(s => s.split(':')) : undefined,
        search: route.query.search != '' || route.query.search ? route.query.search : undefined
      }
      console.log('ezQ.spec',ezQuery.spec) 
      // console.log(ezQuery.spec)
      const pagination = { page: route.query.page || 1, pageSize: 6 };
      const populate = ["category", "images"];
      const sort = route.query.sort ? [route.query.sort] : ["updatedAt:desc"];
      const search = route.query.search != '' && route.query.search ? route.query.search : undefined
      const spec = route.query.spec
      console.log(spec)

      const filters = {
        $and: [
          {
            price: {
              $gte: route.query.price_gte,
            },
          },
          {
            price: {
              $lte: route.query.price_lte,
            },
          },
          {
            category: {
              id: {
                $eq: route.query.categoryId || categories[0]?.id,
              }
            }
          },
          {
            title: {
              $containsi: search,
            }
          },
        ],
      };
      if (ezQuery.spec) {
        ezQuery.spec.map(s => {
          filters.$and.push({
            spec: {
              $containsi: `"${s[0]}":${s[1]}`
            }
          })
        })
      }
      const query = {
        populate,
        pagination,
        sort,
        filters,
      }
      await eCommerceStore.fetchCategories();
      await eCommerceStore.fetchProducts(query);
      console.log(filters);
    }
    watchEffect(() => {
      createProductQuery();
    });
    onMounted(() => { });
    return {
      products,
      eCommerceStore,
      pushQueryToUrl,
      categories,
      sort,
      asd,
      searchString
    };
  },
  name: "Products",
};
</script>

<style scoped>
.body {
width: 1200px;
margin: 0 auto;
background-color: #fff;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0px 20px;
}
.navigation {
  display: flex;
  align-items: center;
}
.navigation li {
  list-style-type: none;
}
.navigation li:hover {
  color: #A18A68;
  text-decoration: underline;
}
.navigation input {
  margin-left: 20px;
  padding: 5px;
  border: #A18A68 solid 1px;
  border-radius: 4%;
}
.image-box {
  display: flex;
  align-items: center;
  background-image: url(http://i.yapx.ru/R7rKF.jpg);
  background-size: cover;
  width: 1100px;
  height: 500px;
  padding-left: 100px;
  border-width: 1px 0px;
  border-color: #A18A68;
  border-style: solid;
}
.image-box-text {
  width: 35%;
  font-size: 15px;
}
.image-box-button {
  padding: 10px 20px;
  border: #ffffff solid 0px;
  background-color: #acddc5;
  border-radius: 4%;
  color: #fff;
  font-weight: 700;
  margin: 20px 0px;
  opacity: 0.9;
}
.image-box-button:hover {
  opacity: 1;
}
.sort {
  text-align: right;
  margin: 20px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
}
.sort select {
  padding: 5px;
  border: #A18A68 solid 1px;
  border-radius: 4%;
  color: #757575;
}
.products-and-filters {
  padding: 20px;
  display: flex;
  flex-wrap: wrap;
}
.filters {
  border: 1px solid #A18A68;
  width: 15%;
  padding: 30px;
}
.filters h1 {
  font-size: 24px;
}
.products {
  display: flex;
  flex-wrap: wrap;
  width: 79%;
}

.products img {
  width: 200px;
}
.product-img {
  display: flex;
  justify-content: center;
}
.price {
  color:#A18A68;
  font-weight: 700;
}
.product-card {
  width:263px;
  padding: 20px;
  border: #a18a6859 solid 1px;
}
.product-card:hover {
  border: #A18A68 solid 1px;
  transition: 0.5s;
}
.pages {
  display: flex;
  margin-bottom: 10px;
}
.pages button {
  background-color: transparent;
  color:#A18A68;
  font-weight: 700;
  margin-right: 10px;
  border: none;
}
.footer {
  display: flex;
  align-items: center;
  justify-content: center;
  border-top: 1px solid #a18a6859;
}
.footer a {
  color: #A18A68;
  text-decoration: none;
}
.footer a:hover {
  text-decoration: underline;
}
</style>