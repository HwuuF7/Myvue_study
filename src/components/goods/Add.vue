<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
          title="添加商品信息"
          type="info"
          center
          show-icon :closable="false">
        </el-alert>
        <!-- 步骤条  -0 可以将字符串变为数值-->
        <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
        <!-- 页签 -->
        <el-tabs :tab-position="tabPosition" v-model="activeIndex" :before-leave="beforeTabLeave" @tab-click="tabClicked">
            <el-tab-pane label="基本信息" name="0">
              <el-form-item label="商品名称" prop="goods_name">
                <el-input v-model="addForm.goods_name"></el-input>
              </el-form-item>
              <el-form-item label="商品价格" prop="goods_price">
                <!-- 规定输入的数据为数字 -->
                <el-input v-model="addForm.goods_price" type="number"></el-input>
              </el-form-item>
              <el-form-item label="商品重量" prop="goods_weight">
                <el-input v-model="addForm.goods_weight" type="number"></el-input>
              </el-form-item>
              <el-form-item label="商品数量" prop="goods_number">
                <el-input v-model="addForm.goods_number" type="number"></el-input>
              </el-form-item>
              <el-form-item label="商品分类" prop="goods_cat">
                <el-cascader expand-trigger="hover" :options="cateList" :props="cateProps" v-model="addForm.goods_cat" @change="handleChanged"></el-cascader>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品参数" name="1">
              <el-form-item :label="item.attr_name" v-for="item in activeData" :key="item.attr_id">
                <!-- 复选框组 -->
                <el-checkbox-group v-model="item.attr_vals">
                  <el-checkbox border :label="cb" v-for="(cb,i) in item.attr_vals" :key="i"></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品属性" name="2">
              <el-form-item :label="item.attr_name" v-for="item in staticData" :key="item.attr_id">
                <el-input v-model="item.attr_vals"></el-input>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品图片" name="3">
              <el-upload
                :action="upURL"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                list-type="picture"
                :headers="headObj"
                :on-success="handleSuccess">
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
            </el-tab-pane>
            <el-tab-pane label="商品内容" name="4">
              <quill-editor v-model="addForm.goods_introduce"></quill-editor>
              <el-button type="primary" class="btnADD" @click="addShop">添加商品</el-button>
            </el-tab-pane>
          </el-tabs>
          </el-form>
    </el-card>
    <el-dialog
      title="图片预览"
      :visible.sync="preDialogVisible"
      width="50%">
      <img :src="prePath"  class="preImg" />
    </el-dialog>

  </div>
</template>

<script>
export default {
  data() {
    return {
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      activeIndex: '0',
      tabPosition: 'left',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      // goods_catNew: '',
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      activeData: [],
      staticData: [],
      upURL: 'https://www.liulongbin.top:8888/api/private/v1/upload',
      headObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      prePath: '',
      preDialogVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取失败！')
      this.cateList = res.data
    },
    handleChanged() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请选择分类！！')
        return false
      }
    },
    async tabClicked() {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateThirdId}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取失败')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.activeData = res.data
        console.log(this.activeData)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateThirdId}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取失败')
        // res.data.forEach(item => {
        //   item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        // })
        this.staticData = res.data
        console.log(this.staticData)
      }
    },
    handlePreview(file) {
      this.prePath = file.response.data.url
      this.preDialogVisible = true
    },
    // 移出图片
    handleRemove(file) {
      const fpath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex(x => x.pic === fpath)
      this.addForm.pics.splice(index, 1)
      console.log(this.addForm.pics)
    },
    handleSuccess(response) {
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    addShop() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的信息！')
        var form = JSON.parse(JSON.stringify(this.addForm))
        form.goods_cat = this.addForm.goods_cat.join(',')
        this.activeData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        this.staticData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        // console.log(form)
        const { data: res } = await this.$http.post('/goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    // 三级分类ID
    cateThirdId() {
      if (this.addForm.goods_cat.length === 3) return this.addForm.goods_cat[2]
      return null
    }
  }
}
</script>

<style scoped>
.el-checkbox {
  margin: 0 7px 0 0 !important;
}
.preImg {
  width: 100%;
}
.btnADD {
  margin: 10px 0;
}
</style>
