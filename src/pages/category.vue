<script setup lang="ts">
import {ref} from "vue";
import {api} from "boot/axios";

const  tab =ref(1);
const  splitterModel = ref(20);

const categoryList=ref([]);
async function getCategoryInfo() {
  const response = await api.get('/category/categoryInfo?parentId=')
  const data = response.data;
  if (data.code == 200) {
    categoryList.value = data.data
    getChildCategoryInfo(categoryList.value[0].categoryId);
  }
}
// prodCount();
getCategoryInfo();
const childCategoryList=ref([]);
async function getChildCategoryInfo(parentId:number) {
  const response = await api.get(`/category/categoryInfo?parentId=${parentId}`)
  const data = response.data;
  if (data.code == 200) {
    childCategoryList.value = data.data
  }
}

function changeTab(){
  getChildCategoryInfo(tab.value);
}
function getImageUrl(imgUrl:string) {
  return `https://image.51x.uk/blackwhite${imgUrl}`;
}
// prodCount();
// getChildCategoryInfo();
</script>

<template>
  <div>
    <q-splitter
        v-model="splitterModel"
        style="height: 100vh"
    >

      <template v-slot:before>
        <q-tabs
            v-model="tab"
            vertical
            class="text-teal"
            @update:model-value="changeTab"
        >

          <q-tab v-for="(item, index) in categoryList" :key="index"  :name="item.categoryId"
                 :label="item.categoryName" />
<!--          name<q-tab name="alarms" icon="alarm" label="Alarms" />-->
<!--          <q-tab name="movies" icon="movie" label="Movies" />-->
        </q-tabs>
      </template>

      <template v-slot:after>
        <q-tab-panels
            class="bg-grey-2"
            v-model="tab"
            animated
            swipeable
            vertical
            transition-prev="jump-up"
            transition-next="jump-up"
            style="height: 100vh"
        >
          <q-tab-panel v-for="(item, index) in categoryList" :key="index" :name="item.categoryId" >
            <div   class="rounded-borders  div-center-child"  >
                <div class="row justify-center q-gutter-sm">
                  <div
                      v-for="(value,index) in childCategoryList"
                      :key="index"
                  >
                    <q-card flat bordered class="q-ma-xs m-category-card">
                      <img :src="getImageUrl(value.pic)"
                           class="m-category-card-image"
                      >
                      <q-card-section class="text-center">
                        <q-item-label  >
                          <div class="text-h6">
                            <a :href="'/product/order?categoryId=' + value.categoryId">
                            <p class="text-caption two-line-clamp tight-spacing">{{value.categoryName }}</p>
                            </a>
                          </div>
                        </q-item-label>
                      </q-card-section>
                    </q-card>

                  </div>
                </div>
            </div>

          </q-tab-panel>
        </q-tab-panels>
      </template>

    </q-splitter>
  </div>
</template>
<style scoped>
.m-category-card{
  height: 160px;
  width: 100px;
}
.m-category-card-image{
  height: 120px;
  width: 100px;
}
</style>
