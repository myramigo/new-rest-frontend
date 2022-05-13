<template>
  <div class="body">
    <div class="header">
      <h1>Products</h1>
      <div class="navigation">
        <div class="categories" v-for="c in categories.data" :key="c.id">
          <ul>
            <li @click="pushQueryToUrl({categoryId: c.id})">{{ c.title }}</li>
          </ul>
        </div>
      <input type="search" placeholder="search" v-model="searchString" @input="pushQueryToUrl({search: searchString})" />
      </div>
    </div>
    <div class="image">
 <img src="https://cdn0.ipoint.kz/AfrOrF3gWeDA6VOlDG4TzxMv39O7MXnF4CXpKUwGqRM/resize:fit:1400:600:0/q:100/plain/s3://complex-images/29/3184/2100-ru-1.png"/>
  </div>
    <div class="sort">
    <select name="sort" v-model="sort" @change="asd($event)">
      <option value="updatedAt:desc">Newest</option>
      <option value="price:asc">Price: Ascending</option>
      <option value="price:desc">Price: Descending</option>
    </select>
    </div>
    <div class="products">
      <div class="product-card" v-for="p in products.data" :key="p.id">
        <div class="product-img"><img src="https://cdn0.ipoint.kz/AfrOrF3gWeDA6VOlDG4TzxMv39O7MXnF4CXpKUwGqRM/resize:fill:540/bg:f6f6f6/q:100/plain/s3://catalog-products/220309095622598612/220311150021758242.png"/></div>
        <p>{{ p.title }}</p>
        <p>{{ p.price }}</p>
      </div>
    </div>
    <hr>
    <div class="pages">
    <div v-for="i in products?.meta?.pagination?.pageCount" :key="i">
      <button @click="pushQueryToUrl({page: i})">{{ i }}</button>
    </div>
    </div>
    <!-- <div>
       <input type="checkbox"   value='lidar' @change="pushQueryToUrl({spec: $event.target.value})"><label>lidar</label>

    </div> -->
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
          ezQuery[key] = value;
        }
        else{
          ezQuery[key] = undefined;
        }
      });
      
      router.push({query: ezQuery})
    }
    async function createProductQuery() {
      ezQuery = {
        page: route.query.page,
        gte: route.query.gte,
        lte: route.query.lte,
        sort: route.query.sort,
        categoryId: route.query.categoryId,
        spec: route.query.spec ? route.query.spec.split(',').map(s => s.split(':')) : undefined,
        search: route.query.search != '' || route.query.search ? route.query.search : undefined
      }
      console.log(ezQuery.spec)
      const pagination = {page: route.query.page || 1, pageSize: 6};
      const populate = ["category"];
      const sort = route.query.sort ? [route.query.sort] : ["updatedAt:desc"];
      const search = route.query.search != '' && route.query.search ? route.query.search : undefined
      const spec = route.query.spec
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
              category:{
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
    onMounted(() => {});    
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
padding: 20px;
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
.image {
  display: flex;
  justify-content: center;
  padding-bottom: 20px;
}
.image img {
  width: 90%;
}
.sort {
  text-align: right;
  margin: 0px 20px;
}
.products img {
  width: 270px;
}
.product-img {
  display: flex;
  justify-content: center;
}
.products {
  padding: 20px;
  display: flex;
  justify-content:center;
  flex-wrap: wrap;
}
.product-card {
  width:300px;
  /* text-align: center; */
  padding: 10px;
  border: transparent solid 1px;
}
.product-card:hover {
  border: #A18A68 solid 1px;
  transition: 0.5s;
}
.pages {
  display: flex;
}
.pages button {
  background-color: transparent;
  font-weight: 600;
  margin-right: 10px;
  border: none;
}
</style>