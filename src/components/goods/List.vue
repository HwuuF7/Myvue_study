<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入:" v-model="queryInfo.query" clearable @clear="getList">
            <el-button slot="append" icon="el-icon-search" @click="getListSnd"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAdd">添加商品</el-button>
        </el-col>
      </el-row>
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="90px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template v-slot:default="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template v-slot:default="scope">
            <el-button type="primary" icon="el-icon-edit"></el-button>
            <el-button type="danger" icon="el-icon-delete" @click='deleteById(scope.row)'></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[5, 7, 10, 12]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total" background>
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 7
      },
      goodslist: [],
      total: 0
    }
  },
  created() {
    this.getList()
  },
  methods: {
    async getList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      // this.$message.success('获取成功')
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getList()
    },
    getListSnd() {
      this.queryInfo.pagenum = 1
      this.getList()
    },
    async deleteById(row) {
      const confirmRes = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmRes !== 'confirm') return this.$message.info('已取消操作！')
      const { data: res } = await this.$http.delete(`goods/${row.goods_id}`)
      if (res.meta.status !== 200) return this.$message.error('删除无效')
      this.$message.success('删除成功！')
      this.getList()
    },
    goAdd() {
      this.$router.push('/goods/add')
    }
  }
}
</script>

<style scoped>
.el-table {
  margin: 15px 0;
}
</style>
