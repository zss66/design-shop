<template>
  <div class="add">
    <el-card class="add-container">
      <el-form :model="state.goodForm" :rules="state.rules" ref="goodRef" label-width="100px" class="goodForm">
        <el-form-item required label="商品分类">
          <el-cascader :placeholder="state.defaultCate" style="width: 300px" :props="state.category" @change="handleChangeCate"></el-cascader>
        </el-form-item>
        <el-form-item label="商品名称" prop="goodsName">
          <el-input style="width: 300px" v-model="state.goodForm.goodsName" placeholder="请输入商品名称"></el-input>
        </el-form-item>
        <el-form-item label="商品简介" prop="goodsIntro">
          <el-input style="width: 300px" type="textarea" v-model="state.goodForm.goodsIntro" placeholder="请输入商品简介(100字)"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="originalPrice">
          <el-input type="number" min="0" style="width: 300px" v-model="state.goodForm.originalPrice" placeholder="请输入商品价格"></el-input>
        </el-form-item>
        <el-form-item label="商品售卖价" prop="sellingPrice">
          <el-input type="number" min="0" style="width: 300px" v-model="state.goodForm.sellingPrice" placeholder="请输入商品售价"></el-input>
        </el-form-item>
        <el-form-item label="商品库存" prop="stockNum">
          <el-input type="number" min="0" style="width: 300px" v-model="state.goodForm.stockNum" placeholder="请输入商品库存"></el-input>
        </el-form-item>
        <el-form-item label="中餐专用标记" prop="tag">
          <el-radio-group v-model="state.goodForm.tag">
      <el-radio :value=0 size="large" border>中式素菜</el-radio>
      <el-radio :value=1 size="large" border>中式荤菜</el-radio>
      <el-radio :value=2 size="large" border>其他</el-radio>
   
    </el-radio-group>
        </el-form-item>
        <el-form-item label="上架状态" prop="goodsSellStatus">
          <el-radio-group v-model="state.goodForm.goodsSellStatus">
            <el-radio label="0">上架</el-radio>
            <el-radio label="1">下架</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item required label="商品主图" prop="goodsCoverImg">
          <el-upload
            class="avatar-uploader"
            action="http://121.36.193.95:3000/api/uploadimg"
            accept="jpg,jpeg,png"
           
            :show-file-list="false"
            :before-upload="handleBeforeUpload"
            :on-success="handleUrlSuccess"
          >
            <img style="width: 100px; height: 100px; border: 1px solid #e9e9e9;" v-if="state.goodForm.goodsCoverImg" :src="state.goodForm.goodsCoverImg" class="avatar">
            <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
          </el-upload>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitAdd()">{{ state.id ? '立即修改' : '立即创建' }}</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted, getCurrentInstance } from 'vue'
import axios from '@/utils/axios'
import { ElMessage } from 'element-plus'
import { useRoute, useRouter } from 'vue-router'
import { localGet, uploadImgServer, uploadImgsServer } from '@/utils'

const { proxy } = getCurrentInstance()
const editor = ref(null)
const goodRef = ref(null)
const route = useRoute()
const router = useRouter()
const { id } = route.query
const state = reactive({
  uploadImgServer,
  token: localGet('token') || '',
  id: id,
  defaultCate: '',
  goodForm: {
    goodsName: '',
    goodsIntro: '',
    originalPrice: '',
    sellingPrice: '',
    stockNum: '',
    goodsSellStatus: '0',
    goodsCoverImg: '',
    tag: 2
  },
  rules: {
    goodsName: [
      { required: 'true', message: '请填写商品名称', trigger: ['change'] }
    ],
    originalPrice: [
      { required: 'true', message: '请填写商品价格', trigger: ['change'] }
    ],
    sellingPrice: [
      { required: 'true', message: '请填写商品售价', trigger: ['change'] }
    ],
    stockNum: [
      { required: 'true', message: '请填写商品库存', trigger: ['change'] }
    ],
  },
  categoryId: '',
  category: {
    lazy: true,
    lazyLoad(node, resolve) {
      const { level = 0, value } = node
      axios.get('/foo/goods/class', {
        params: {
          shopid:localGet('shoper').id,
          pageNumber: 1,
          pageSize: 1000,
        }
      }).then(res => {
        const list = res.list
        const nodes = list.map(item => ({
          value: item.id,
          label: item.name,
          leaf: level ==0
        }))
        resolve(nodes)
      })
    }
  }
})

onMounted(() => {
  if (id) {
    axios.get(`/foo/goods/detail`,{
      params:{
        id:localGet('shoper').id
      }
    }).then(res => {
      const { goods } = res
      console.log(goods);
      state.goodForm = {
        goodsName: goods.name,
        goodsIntro: goods.dec,
        originalPrice: goods.price,
        sellingPrice: goods.newprice,
        stockNum: goods.offernum,
        goodsSellStatus: String(goods.offer),
        goodsCoverImg: proxy.$filters.prefix(goods.img),
        tag: goods.type
      }
      state.categoryId = goods.classid
      state.defaultCate = goods.classname
    
    })
  }
})

const submitAdd = () => {
  goodRef.value.validate((vaild) => {
    if (vaild) {
      // 默认新增用 post 方法
      let httpOption = axios.post
      let params = {
        classid: state.categoryId,
        img: state.goodForm.goodsCoverImg,
        dec: state.goodForm.goodsIntro,
        name: state.goodForm.goodsName,
        offer: state.goodForm.goodsSellStatus,
        price: state.goodForm.originalPrice,
        newprice: state.goodForm.sellingPrice,
        num: state.goodForm.stockNum,
        type: state.goodForm.tag
      }
      console.log('params', params)
      if (id) {
        params.id = id,
        params.shopid=localGet('shoper').id
      }
      httpOption('/foo/goods/change', params).then(() => {
        ElMessage.success(id ? '修改成功' : '添加成功')
        router.push({ path: '/good' })
      })
    }
  })
}
const handleBeforeUpload = (file) => {
  const sufix = file.name.split('.')[1] || ''
  if (!['jpg', 'jpeg', 'png'].includes(sufix)) {
    ElMessage.error('请上传 jpg、jpeg、png 格式的图片')
    return false
  }
}
const handleUrlSuccess = (val) => {
  state.goodForm.goodsCoverImg = val.data || ''
}
const handleChangeCate = (val) => {
  console.log(val);
  state.categoryId = val[2] || 0
}
</script>

<style scoped>
  .add {
    display: flex;
  }
  .add-container {
    flex: 1;
    height: 100%;
  }
  .avatar-uploader {
    width: 100px;
    height: 100px;
    color: #ddd;
    font-size: 30px;
  }
  .avatar-uploader-icon {
    display: block;
    width: 100%;
    height: 100%;
    border: 1px solid #e9e9e9;
    padding: 32px 17px;
  }
</style>