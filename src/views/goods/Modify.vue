<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>修改商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 提示 -->
      <el-alert title="添加商品信息" type="info" center :closable="false" show-icon></el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200" :active="activeIndex - 0" align-center finish-status="success">
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave" @tab-click="TabClicked">
          <el-tab-pane name="0" label="基本信息">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-if="addForm.goods_cat"
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="{ expandTrigger: 'hover', label: 'cat_name',value: 'cat_id', children: 'children' }"
                @change="handleChange"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane name="1" label="商品参数">
            <!-- 渲染表单的item项 -->
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border :label="cb" v-for="(cb, i) in item.attr_vals" :key="i"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane name="2" label="商品属性">
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane name="3" label="商品图片">
            <el-upload
              :headers="headerObj"
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane name="4" label="商品内容">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button @click="add" type="primary" class="btnAdd">修改商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="50%"
      >
      <img :src="previewPath" alt="" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: "",
  data() {
    return {
      goods_id: '',
      activeIndex: 0,
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: ''
      },
      addFormRules: {
        goods_name: [
          {required: true, message: '请输入商品名称', trigger: 'blur'}
        ],
        goods_price: [
          {required: true, message: '请输入商品价格', trigger: 'blur'}
        ],
        goods_weight: [
          {required: true, message: '请输入商品重量', trigger: 'blur'}
        ],
        goods_number: [
          {required: true, message: '请输入商品数量', trigger: 'blur'}
        ],
        goods_cat: [
          {required: true, message: '请输入商品分类', trigger: 'blur'}
        ],
      },
      cateList: [],
      manyTableData: [],
      onlyTableData: [],
      uploadUrl: 'https://www.liulongbin.top:8888/api/private/v1/upload',
      headerObj:{ Authorization: window.sessionStorage.getItem('token')},
      previewPath: '',
      previewVisible: false
    };
  },

  components: {},
  computed: {
    cateId() {
      if(this.addForm.goods_cat.length == 3) return this.addForm.goods_cat[2]
      return null
    }
  },
  methods: {
    async getGoods() {
      const {data: res} = await this.$http.get(`goods/${this.goods_id}`)
      if(res.meta.status != 200) return this.$message.error('获取商品信息失败')
      console.log(res.data)
      this.addForm.goods_name = res.data.goods_name
      this.addForm.goods_price = res.data.goods_price
      this.addForm.goods_weight = res.data.goods_weight
      this.addForm.goods_number = res.data.goods_number
      for(let i of res.data.goods_cat.split(',')){
        this.addForm.goods_cat.push(parseInt(i))
      }
      this.addForm.goods_introduce = res.data.goods_introduce
      this.addForm.pics = res.data.pics
    },
    async getCateList() {
      const {data: res} = await this.$http.get('categories')
      if(res.meta.status != 200) return this.$message.error('获取数据失败')
      this.cateList = res.data
      // console.log(this.cateList)
    },
    // 级联选择器改变函数
    handleChange() {
      // console.log(this.addForm.goods_cat)
      if(this.addForm.goods_cat.length != 3) this.addForm.goods_cat = []
    },
    beforeTabLeave(activeName, oldActiveName) {
      if(oldActiveName == '0' && this.addForm.goods_cat.length != 3) {
        this.$message.error('请先选择商品分类！')
        return false
      }
    },
    async TabClicked() {
      // 访问的是动态参数面板
      if(this.activeIndex == '1') {
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params: {sel: 'many'}})
        if(res.meta.status != 200) return this.$message.error('获取数据失败!')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length == 0 ? [] : item.attr_vals.split(' ')
        })
        // console.log(res.data)
        this.manyTableData = res.data
      }
      if(this.activeIndex == '2') {
        const {data: res} = await this.$http.get(`categories/${this.cateId}/attributes`, {params: {sel: 'only'}})
        if(res.meta.status != 200) return this.$message.error('获取数据失败！')
        // console.log(res.data)
        this.onlyTableData = res.data
      }
    },
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(item => {
        item.pic == filePath
      })
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm)
    },
    handleSuccess(response) {
      const picInfo = {pic: response.data.tmp_path}
      this.addForm.pics.push(picInfo)
      console.log(this.addForm)
    },
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if(!valid) return this.$message.error('请填写必要内容')
        // lodash clonedeep(obj) 深度拷贝 addForm
        let form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        //发起请求添加商品
        const {data: res} = await this.$http.put(`goods/${this.goods_id}`, form)
        if(res.meta.status != 200) return this.$message.error('修改商品失败')
        this.$message.success('修改成功')
        this.$router.push('/goods')
        // console.log(form)
      })
    },
  },
  created() {
    this.goods_id = this.$route.params.id
    this.getCateList()
    this.getGoods()
  },
  mounted() {},
};
</script>

<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
  .el-steps_title {
    font-size: 14px;
  }
}
.el-checkbox {margin: 0 10px 0 0 !important;}
.previewImg{width: 100%;}
.btnAdd{margin-top: 15px;}
</style>