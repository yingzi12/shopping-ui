<script lang="ts" setup>
import {ref} from "vue";
import {Cookies, useQuasar} from 'quasar'
import {useRouter} from "vue-router";
import {api} from "boot/axios";
import {compressIfNeeded} from "boot/tools";

const token = Cookies.get('token');
const id = Cookies.get('id');
const $q = useQuasar()
const router = useRouter(); // 使用 Vue Router 的 useRouter 函数


const shop = ref({  });
const shopOk = ref(false);

const imgUrl = ref("/favicon.png");

const previewImage = ref(null);
const fileInput = ref<HTMLInputElement | null>(null);
const selectedImage = ref<File | null>(null);
const userHeadImageStr=ref("点击替换Logo");

async function getDetail() {
  const response = await api.get(`/admin/shopDetail/info`);
  const data = response.data;
  if (data.code == 200) {
    shop.value=data.data;
    shopOk.value=true;
    console.log("-----------shop-----------------")
    console.log(shop)
    imgUrl.value = data.data.shopLogo  ;
    previewImage.value =data.data.shopLogo;
  }
}
function triggerFileInput() {
  fileInput.value?.click();
}

async function handleImageUpload(event: Event) {
  try {
    userHeadImageStr.value="上传中....";
    const file = (event.target as HTMLInputElement).files?.[0];
    if (!file) {
      throw new Error("No file selected");
    }
    selectedImage.value = file;
    const compressedFile = await compressIfNeeded(file);
    const formData = new FormData();
    formData.append('file', compressedFile);
    const response = await api.put( '/admin/file/upload',  formData);
    const data = await response.data; // 确保使用 await 等待 json 解析完成
    if (data.code === 200) {
      previewImage.value = $q.config.sourceWeb + data.data;
      imgUrl.value = data.data;
      userHeadImageStr.value="点击替换头像";
    } else {
      userHeadImageStr.value="替换头像失败";
      $q.dialog({
        title: '错误',
        message: data.msg,
        ok: {
          push: true
        },
      }).onOk(async () => {
        console.log("ok");
      });
      // throw new Error('Image upload failed');
    }
  } catch (error) {
    notify('Error uploading image', 'red-5');
  }
}
function notify(message: string, color: string) {
  $q.notify({
    color: color,
    textColor: 'white',
    icon: color === 'red-5' ? 'warning' : 'cloud_done',
    message: message
  });
}
getDetail();
function getImageUrl(url) {
  if (url != null) {
    return `${$q.config.sourceWeb}${url}`;
  }
  return "/favicon.png";
}
const  money=ref(0.0);
const paypalDialog = ref(false);

</script>

<template>
  <div v-if="!shopOk ">
    <q-card>
      <q-card-section>
        <div class="text-h6">待选择店铺</div>
      </q-card-section>
    </q-card>
  </div>
  <div  v-if="shopOk" >
    <router-link   to="/admin/shop/edit">
      <q-btn color="primary" label="编辑店铺信息"/>
    </router-link>
  </div>

  <div  v-if="shopOk"  class="q-pa-md row items-start q-gutter-md">
    <q-card bordered class="my-card" flat>
      <q-item>
        <!--        <q-item-section>-->
        <!--          <q-avatar font-size="52px" size="100px">-->
        <!--            <img :src="previewImage">-->
        <!--          </q-avatar>-->
        <!--        </q-item-section>-->
        <q-item-section @click="triggerFileInput">
          <q-avatar font-size="52px" size="100px">
            <img :src="getImageUrl(previewImage)">
          </q-avatar>
          <input type="file" ref="fileInput" @change="handleImageUpload" hidden>
          <p class="text-caption">{{ userHeadImageStr }}</p>
        </q-item-section>
        <q-item-section>
          <q-item-label>{{ shop.shopName != null ? shop.shopName : '无数据' }}
          </q-item-label>
          <q-item-label v-if="shop.shopId" caption>
            ID:{{ shop.shopId }}
          </q-item-label>
          <q-item-label v-if="shop.shopStatus" caption>
            状态:{{ shop.shopStatus ==-1 ? '未开通' : shop.shopStatus == 0 ? '停业中' : '营业中' }}
          </q-item-label>
        </q-item-section>

      </q-item>
      <q-card-section>
        <q-item-label>手机：{{shop.mobile}}</q-item-label>
        <q-item-label>联系电话：{{shop.tel}}</q-item-label>
        <q-item-label>行业：{{shop.shopIndustry}}</q-item-label>
      </q-card-section>
      <q-separator/>

      <q-card-section  horizontal>
        <div class="text-h6">简介</div>
        <div class="text-body2" style="padding: 10px">
          {{ shop.intro }}
        </div>
      </q-card-section>
      <q-separator/>

      <q-card-section  horizontal>
        <div class="text-h6">广告</div>
        <div class="text-body2" style="padding: 10px">
          {{ shop.shopNotice }}
        </div>
      </q-card-section>
      <q-separator/>

<!--      <q-card-actions>-->
<!--        <q-btn color="red-8" flat icon="favorite" round>{{shop.countAttention }}</q-btn>-->
<!--        <q-btn color="red-8" flat icon="thumb_up" round>{{shop.countLike }}</q-btn>-->
<!--        <q-btn color="red-8" flat icon="visibility" round>{{shop.countSee }}</q-btn>-->
<!--      </q-card-actions>-->
    </q-card>


  </div>
  <q-dialog v-model="paypalDialog">
    <PayaplCard :amount="money" :productId="shop.id" :kind="5" intro="充值余额" productName="充值余额" url='/users'/>
  </q-dialog>
</template>

<style lang="sass" scoped>
.my-card
  width: 100%
  max-width: 350px
</style>
