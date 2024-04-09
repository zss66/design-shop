<template>
  <div class="header">
    <div class="left">
      <el-icon class="back" v-if="state.hasBack" @click="back"><Back /></el-icon>
      <span style="font-size: 20px">{{ state.name }}</span>
    </div>
    <div class="right">
      <el-popover
        placement="bottom"
        :width="320"
        trigger="click"
        popper-class="popper-user-box"
      >
        <template #reference>
          <div class="author">
            {{ state.userInfo && state.userInfo.nickname || '' }}
            <el-icon><CaretBottom /></el-icon>
          </div>
        </template>
        <div class="nickname">
          <div style="display:flex ;align-items: center;">
            <p style="line-height: 40px;">头像： </p>
          <el-avatar
        :src="state.userInfo.showurl"
      />
          </div>
         
         
          <p>昵称：{{ state.userInfo && state.userInfo.nickname || '' }}</p>
          <el-tag size="small" effect="dark" class="logout" @click="logout">退出</el-tag>
        </div>
      </el-popover>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive } from 'vue'
import { useRouter } from 'vue-router'
import axios from '@/utils/axios'
import { localRemove, pathMap,localGet } from '@/utils'

const router = useRouter()
const state = reactive({
  name: '商家后台',
  userInfo: '', // 用户信息变量
  hasBack: false, // 是否展示返回icon
})
// 初始化执行方法
  onMounted(() => {
  const pathname = window.location.hash.split('/')[1] || ''
  if (!['login'].includes(pathname)) {
      state.userInfo=localGet('shoper')
  }
})

// 退出登录
const logout = () => {
   // 退出之后，将本地保存的 token  清理掉
   localRemove('shoper')
    // 回到登录页
    router.push({ path: '/login' })
}

router.afterEach((to) => {
  const { id } = to.query
  state.name = pathMap[to.name]
  // level2 和 level3 需要展示返回icon
  console.log('to.name', to.name)
  state.hasBack = ['level2', 'level3'].includes(to.name)
})

// 返回方法
const back = () => {
  router.back()
}
</script>

<style scoped>
  .header {
    height: 50px;
    border-bottom: 1px solid #e9e9e9;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
  }
  .header .left .back {
    border: 1px solid #e9e9e9;
    padding: 5px;
    border-radius: 50%;
    margin-right: 5px;
    cursor: pointer;
  }
  .right > div > .icon{
    font-size: 18px;
    margin-right: 6px;
  }
  .author {
    margin-left: 10px;
    cursor: pointer;
    display: flex;
    align-items: center;
    
  }
</style>

<style>
  .popper-user-box {
    background: url('https://s.yezgea02.com/lingling-h5/static/account-banner-bg.png') 50% 50% no-repeat!important;
    background-size: cover!important;
    border-radius: 0!important;
  }
   .popper-user-box .nickname {
    position: relative;
    color: #ffffff;
  }
  .popper-user-box .nickname .logout {
    position: absolute;
    right: 0;
    top: 0;
    cursor: pointer;
  }
</style>