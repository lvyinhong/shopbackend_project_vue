<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图-->
    <el-card>
      <el-row>
        <el-col>
          <!--添加角色按钮区域-->
          <el-button
            type="primary"
            size="mini"
          >添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <el-table
        :data="rolesList"
        border
        stripe
      >
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0?'bdtop':'', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!--渲染一级权限-->
              <el-col :span="5">
                <el-tag> {{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--渲染二级和三级权限权限-->
              <el-col :span="19">
                <!--通过for循环嵌套渲染二级权限-->
                <el-row
                  :class="[i2 === 0?'':'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag type="success"> {{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>

              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column
          type="index"
          label="#"
        ></el-table-column>
        <el-table-column
          label="角色名称"
          prop="roleName"
        ></el-table-column>
        <el-table-column
          label="角色描述"
          prop="roleDesc"
        >
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
            >删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolesList: []
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      console.log(res)
    },
    // 根据id删除对应的权限
    async removeRightById() {
      const confirmResult = await this.$confirm('此操作将永久删除该角色下的权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除权限')
      }
      console.log(confirmResult)
    }
  }
}
</script>
<style scoped lang='less'>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eeeeee;
}
.bdbottom {
  border-bottom: 1px solid #eeeeee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
