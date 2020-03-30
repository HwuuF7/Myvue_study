<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot:default="scope">
            <el-row :class="['borbottom', i1 === 0 ? 'bortop': '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key='item1.id'>
              <!-- 一级权限 -->
              <el-col :span="6">
                <el-tag closable @close="removeTag(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限 -->
              <el-col :span="18">
                <el-row :class="[i2 === 0 ? '' : 'bortop', 'vcenter']" v-for="(item2,i2) in item1.children" :key='item2.id'>
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeTag(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="item3 in item2.children" :key='item3.id' type="warning" closable @close="removeTag(scope.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色" prop="roleName"></el-table-column>
        <el-table-column label="描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="scope">
           <!-- console.log(scope.row) -->
            <el-button type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button type="danger" icon='el-icon-delete'>删除</el-button>
            <el-button type="warning" icon='el-icon-setting' @click='setRightDialog(scope.row)'>分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <el-tree :data="rightslist" :props="rightsTreeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="rightsKey" ref='rightsRef'></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data() {
    return {
      rolelist: [],
      setRightDialogVisible: false,
      // 权限数据
      rightslist: [],
      rightsTreeProps: {
        label: 'authName',
        children: 'children'
      },
      // 分配权限绑定的三级节点勾选
      rightsKey: [],
      // 分配权限时捕获当前角色ID
      roleID: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.rolelist = res.data
    },
    async removeTag(role, rightId) {
      console.log(role)
      const confirmRes = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmRes !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      // this.getRolesList()
      role.children = res.data
      this.$message.success('删除成功')
    },
    async setRightDialog(role) {
      this.roleID = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败')
      this.rightslist = res.data
      console.log(this.rightslist)
      this.getRightsKey(role, this.rightsKey)
      this.setRightDialogVisible = true
    },
    // 获取已勾选权限节点
    getRightsKey(node, arr) {
      if (!node.children) return arr.push(node.id)
      node.children.forEach(item => this.getRightsKey(item, arr))
    },
    // 监听权限对话框关闭
    setRightDialogClosed() {
      this.rightsKey = []
    },
    async allotRights() {
      const keys = [...this.$refs.rightsRef.getCheckedKeys(), ...this.$refs.rightsRef.getHalfCheckedKeys()]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleID}/rights`, { rids: idStr })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('分配失败')
      this.$message.success('分配成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style scoped>
.el-table{
    margin-top: 15px;
    font-size: 12px;
}
.el-tag {
  margin: 7px;
}
.bortop {
  border-top: 1px solid #eee;
}
.borbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
