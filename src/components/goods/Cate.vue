<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card class="box-card">
      <el-row>
        <el-col>
          <el-button
            type="primary"
            @click="showAddCateDialog"
          >添加分类</el-button>
        </el-col>
      </el-row>
      <!--表格-->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!--是否有效-->
        <template
          slot="isok"
          slot-scope="scope"
        >
          <i
            style="color:lightgreen;"
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i
            style="color:red;"
            class="el-icon-error"
            v-else
          ></i>
        </template>

        <template
          slot="order"
          slot-scope="scope"
        >
          <el-tag
            size="mini"
            v-if="scope.row.cat_level === 0"
          >一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
          >二级</el-tag>
          <el-tag
            type="warning"
            size="mini"
            v-else
          >三级</el-tag>
        </template>
        <template
          slot="opt"
          slot-scope="scope"
        >
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
          ></el-button>
        </template>
      </tree-table>
      <!--分页区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1,2,5,10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!--添加分类的对话框-->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisiable"
      width="30%"
      @close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item
          label="分类名称"
          prop="cat_name"
        >
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!--级联选择框-->
          <!--props用来指定具体的配置对象-->
          <el-cascader
            :options="parentCateList"
            :props="cascaderProps"
            v-model="selectedKeys"
            @change="parentCateChanged()"
            clearable
            :change-on-select="true"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addCateDialogVisiable = false">取 消</el-button>
        <el-button
          type="primary"
          @click="addCate()"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类的数据列表，默认为空
      catelist: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // 为table 指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示当前定义列为模版列
          type: 'template',
          // 表示当前一列使用模版名称
          template: 'isok'
        },
        {
          label: '排序',
          // 表示当前定义列为模版列
          type: 'template',
          // 表示当前一列使用模版名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示当前定义列为模版列
          type: 'template',
          // 表示当前一列使用模版名称
          template: 'opt'
        }
      ],
      addCateForm: {
        cat_pid: 0, // 父级分类id
        cat_name: '', // 分类名称
        cat_level: 0 // 分类的等级，默认要添加的是一级分类
      },
      addCateDialogVisiable: false,
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类的名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 级联选择器的选择结果
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('/categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.catelist = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize改变的时间
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange(currentPage) {
      this.queryInfo.pagenum = currentPage
      this.getCateList()
    },
    // 点击按钮展示添加分类的对话框
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisiable = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类失败')
      this.parentCateList = res.data
    },
    // 选择项发生变化处罚该函数
    parentCateChanged() {
      // 如果selectedKeys 数组的len大于0证明选中了父级分类，
      // 反之没有选择任何父级分类
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 当前分类的等级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击确定,添加新的分类
    async addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (valid) { return false }
      })
      const { data: res } = await this.$http.post('categories', this.addCateForm)
      console.log(res)
      if (res.meta.status !== 201) {
        return this.$message.error('添加新分类失败')
      }
      this.$message.success('添加新分类成功')
      this.getCateList()
      this.addCateDialogVisiable = false
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  }
}
</script>
<style scoped lang="less">
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>>
