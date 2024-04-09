<template>
  <el-card class="account-container">
    <el-form :model="state.nameForm" :rules="state.rules" ref="nameRef" label-width="150px" label-position="right" class="demo-ruleForm">
      <el-form-item label="登录名：" prop="loginName">
        <el-input style="width: 200px" v-model="state.nameForm.loginName"></el-input>
      </el-form-item>
      <el-form-item label="昵称：" prop="nickName">
        <el-input style="width: 200px" v-model="state.nameForm.nickName"></el-input>
      </el-form-item>
      <el-form-item label="头像：" prop="showurl">
        <el-upload
    class="avatar-uploader"
    action="http://121.36.193.95:3000/api/uploadimg"
            accept="jpg,jpeg,png"
    :show-file-list="false"
    :on-success="handleAvatarSuccess"
    :before-upload="beforeAvatarUpload"
  >
    <img v-if="state.nameForm.showurl" :src="state.nameForm.showurl" class="avatar" />
    <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
  </el-upload>
       
      </el-form-item>
      <el-form-item label="商铺标题：" prop="title">
        <el-input style="width: 200px" v-model="state.nameForm.title"></el-input>
      </el-form-item>
      <el-form-item label="展示横图：" prop="swiperurl">
        <el-upload
    class="avatar-uploader"
    action="http://121.36.193.95:3000/api/uploadimg"
            accept="jpg,jpeg,png"
    :show-file-list="false"
    :on-success="handleAvatarSuccess"
    :before-upload="beforeAvatarUpload"
  >
    <img v-if="state.nameForm.swiperurl" :src="state.nameForm.swiperurl" class="avatar" />
    <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
  </el-upload>
       
      </el-form-item>
      <el-form-item label="商铺类型：" prop="type">
        <el-radio-group v-model="state.nameForm.type">
      <el-radio value="中式快餐" size="large" border>中式快餐</el-radio>
      <el-radio value="西式快餐" size="large" border>西式快餐</el-radio>
      <el-radio value="其他精品" size="large" border>其他精品</el-radio>
    </el-radio-group>
      </el-form-item>
      <el-form-item label="商铺介绍：" prop="shopdec">
        <el-input
    v-model="state.nameForm.shopdec"
    maxlength="50"
    style="width: 400px"
    placeholder="Please input"
    show-word-limit
    type="textarea"
  />
        
      </el-form-item>
      <el-form-item label="提供服务类型：" prop="offer">
        <el-checkbox-group v-model="state.nameForm.offer">
    <el-checkbox label="可堂食" value="可堂食" />
    <el-checkbox label="可打包" value="可打包" />
    <el-checkbox label="可预约" value="可预约" />
    <el-checkbox label="隐藏预约" value="隐藏预约" />
    
  </el-checkbox-group>
        
      </el-form-item>
      <el-form-item label="商铺地址：" prop="address">
        <el-input
    v-model="state.nameForm.address"
    maxlength="50"
    style="width: 240px"
    placeholder="Please input"
    show-word-limit
    type="textarea"
  />
        
      </el-form-item>
      <el-form-item label="联系电话：" prop="phone">
        <el-input style="width: 200px" v-model="state.nameForm.phone"></el-input>
      </el-form-item>
      <el-form-item label="营业时间：" prop="opentime">
        <el-input style="width: 200px" v-model="state.nameForm.opentime"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="danger" @click="submitName">确认修改</el-button>
      </el-form-item>
    </el-form>
  </el-card>
  <el-card class="account-container">
    <el-form :model="state.passForm" :rules="state.rules" ref="passRef" label-width="150px" label-position="right" class="demo-ruleForm">
      <el-form-item label="原密码：" prop="oldpass">
        <el-input style="width: 200px" v-model="state.passForm.oldpass"></el-input>
      </el-form-item>
      <el-form-item label="新密码：" prop="newpass">
        <el-input style="width: 200px" v-model="state.passForm.newpass"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="danger" @click="submitPass">确认修改</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue'
import axios from '@/utils/axios'
import { ElMessage } from 'element-plus'
import md5 from 'js-md5'
import { Plus } from '@element-plus/icons-vue'
import { localGet } from '@/utils'
const nameRef = ref(null)
const passRef = ref(null)
const state = reactive({
  user: null,
  nameForm: {
    loginName: '',
    nickName: '',
    showurl:'',
    title:'',
    swiperurl:'',
    type:'',
    shopdec:'',
    offer:[],
    address:'',
    phone:'',
    opentime:''

  },
  passForm: {
    oldpass: '',
    newpass: ''
  },
  rules: {
    loginName: [
      { required: 'true', message: '登录名不能为空', trigger: ['change'] }
    ],
    nickName: [
      { required: 'true', message: '昵称不能为空', trigger: ['change'] }
    ],
    oldpass: [
      { required: 'true', message: '原密码不能为空', trigger: ['change'] }
    ],
    newpass: [
      { required: 'true', message: '新密码不能为空', trigger: ['change'] }
    ]
  },
})
onMounted(() => {
  axios.get('/foo/shoper/info', {
  params: {
    id: localGet('shoper').id
  }
}).then(res => {
 console.log(res);
 state.nameForm.loginName=res[0].loginname
 state.nameForm.address =res[0].address;
state.nameForm.nickName=res[0].nickname;
state.nameForm.offer=res[0].offer.split(',');
 state.nameForm.opentime=res[0].opentime;
 state.nameForm.phone=res[0].phone;
 state.nameForm.shopdec=res[0].shopdec;
state.nameForm.showurl=res[0].showurl;
state.nameForm.swiperurl=res[0].swiperurl;
 state.nameForm.title=res[0].title;
 state.nameForm.type=res[0].type.toString();
console.log(typeof state.nameForm.offer);
}).catch(error => {
  // 处理异常情况
  console.error('请求出错', error);
});
})
const submitName = () => {
  nameRef.value.validate((vaild) => {
    if (vaild) {
      axios.put('/foo/shoper/changeinfo', {
        id:localGet('shoper').id,
        data:state.nameForm
      }).then(() => {
        ElMessage.success('修改成功')
        window.location.reload()
      })
    }
  })
}
const submitPass = () => {
  passRef.value.validate((vaild) => {
    if (vaild) {
      axios.put('/foo/shoper/pws', {
        id:localGet('shoper').id,
        oldpws: md5(state.passForm.oldpass),
        newpws: md5(state.passForm.newpass)
      }).then(() => {
        ElMessage.success('修改成功')
        window.location.reload()
      })
    }
  })
}


const imageUrl = ref('')

    const handleAvatarSuccess = (response, uploadFile) => {
      imageUrl.value = URL.createObjectURL(uploadFile.raw)
    }

    const beforeAvatarUpload = (rawFile) => {
      if (rawFile.size / 1024 / 1024 > 2) {
        ElMessage.error('Avatar picture size can not exceed 2MB!')
        return false
      }
      return true
    }

</script>

<style>
  .account-container {
    margin-bottom: 20px;
  }
</style>
<style scoped>
.avatar-uploader .avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
</style>