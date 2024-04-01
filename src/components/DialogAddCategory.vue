<template>
  <el-dialog
    :title="state.type == 'add' ? '添加分类' : '修改分类'"
    v-model="state.visible"
    width="400px"
  >
    <el-form :model="state.ruleForm" :rules="state.rules" ref="formRef" label-width="100px" class="good-form">
      <el-form-item label="商品名称" prop="name">
        <el-input type="text" v-model="state.ruleForm.name"></el-input>
      </el-form-item>
      <el-form-item label="排序值" prop="rank">
        <el-input type="number" v-model="state.ruleForm.rank"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="state.visible = false">取 消</el-button>
        <el-button type="primary" @click="submitForm">确 定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { useRoute } from 'vue-router'
import axios from '@/utils/axios'
import { ElMessage } from 'element-plus'

const props = defineProps({
  type: String, // 用于判断是添加还是编辑
  reload: Function // 添加或修改完后，刷新列表页
})

const formRef = ref(null)
const route = useRoute()
const state = reactive({
  visible: false,
  ruleForm: {
    name: '',
    rank: ''
  },
  rules: {
    name: [
      { required: 'true', message: '名称不能为空', trigger: ['change'] }
    ],
    rank: [
      { required: 'true', message: '编号不能为空', trigger: ['change'] }
    ]
  },
  id: ''
})
// 获取详情
const getDetail = (id) => {
  axios.get(`/foo/goods/class`,{
    params: {
    id
  }
  }).then(res => {
    state.ruleForm = {
       name: res[0].name,
       rank: res[0].order
    }
    
    
  }) .catch(error => {
    console.error('发生错误：', error);
  });
}
// 开启弹窗
const open = (id) => {
  state.visible = true
  if (id) {
    state.id = id
    // 如果是有 id 传入，证明是修改模式
    getDetail(id)
  } else {
    // 否则为新增模式
    // 新增类目
    state.ruleForm = {
      name: '',
      rank: ''
    }
    
   
  }
}
// 关闭弹窗
const close = () => {
  state.visible = false
}
const submitForm = () => {
  formRef.value.validate((valid) => {
    if (valid) {
      if (props.type == 'add') {
        // 添加方法
        axios.post('/foo/goods/classedit', {
          name: state.ruleForm.name,
          order: state.ruleForm.rank
        }).then(() => {
          ElMessage.success('添加成功')
          state.visible = false
          // 接口回调之后，运行重新获取列表方法 reload
          if (props.reload) props.reload()
        })
      } else {
        // 修改方法
        axios.post('/foo/goods/classedit', {
          id: state.id,
          name: state.ruleForm.name,
          order: state.ruleForm.rank
        }).then(() => {
          ElMessage.success('修改成功')
          state.visible = false
          // 接口回调之后，运行重新获取列表方法 reload
          if (props.reload) props.reload()
        })
      }
    }
  })
}
defineExpose({ open, close })
</script>