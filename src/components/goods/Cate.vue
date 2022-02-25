<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <el-row>
        <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      </el-row>
      <tree-table
        class="cateListClass"
        :data="cateList"
        :columns="columnsProp"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false">
        <!-- 是否有效列 -->
        <template slot="isOk" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: rgb(63, 148, 105); font-size: 20px"></i>
          <i class="el-icon-error" style="color: red; font-size: 20px;" v-else></i>
        </template>
        <!-- 等级列 -->
        <template slot="order" slot-scope="scope">
          <el-tag type v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作列 -->
        <!-- TODO -->
        <template slot="operation" slot-scope="scope" class="operateClass" >
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          <el-button type="primary" size="small" round @click="showEditCateDialog(scope.row)">编辑</el-button>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          <el-button type="danger" size="small" round @click="deleteCategoryById(scope.row)">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background>
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="40%"
      @closed="closeAddCateDialog">
      <!-- 填加分类表单 -->
      <el-form 
        ref="addCateFormRef" 
        :model="addCateFormData" 
        label-width="90px"
        :rules="addCateFormRules">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateFormData.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader
            v-model="selectedParentCateIds"
            :options="parentCateList"
            :props="{ 
              expandTrigger: 'hover',
              label: 'cat_name',
              value: 'cat_id',
              children: 'children',
              checkStrictly: true
            }"
            clearable
            @change="handleChange">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="40%">
      <el-form 
        ref="EditCateFormRef" 
        :model="editCateFormData" 
        label-width="80px"
        :rules="EditCateFormRules">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateFormData.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitEditCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created() {
    this.getCateList();
  },
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      cateList: [],
      addCateDialogVisible: false,
      addCateFormData: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      parentCateList: [],
      selectedParentCateIds: [],
      // 树形表格，列属性配置项
      columnsProp: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isOk'
      }, {
        label: '排序',
        type: 'template',
        template: 'order'
      }, {
        label: '操作',
        type: 'template',
        template: 'operation'
      }],
      addCateFormRules: {
        cat_name: { required: true, message: '请输入分类名称', trigger: 'blur' }
      },
      editCateDialogVisible: false,
      editCateFormData: {},
      EditCateFormRules: {
        cat_name: { required: true, message: '请输入分类名称', trigger: 'blur' }
      }
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    showAddCateDialog() {
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.parentCateList = res.data;
    },
    handleChange() {
      if (this.selectedParentCateIds.length < 1) {
        this.addCateFormData.cat_pid = 0;
        this.addCateFormData.cat_level = 0;
        return;
      }
      this.addCateFormData.cat_pid = this.selectedParentCateIds.slice(-1)[0];
      this.addCateFormData.cat_level = this.selectedParentCateIds.length;
    },
    closeAddCateDialog() {
      this.$refs.addCateFormRef.resetFields();
      this.addCateFormData.cat_level = 0;
      this.addCateFormData.cat_pid = 0;
      this.selectedParentCateIds = [];
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post('categories', this.addCateFormData);
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg);
        this.$message.success(res.meta.msg);
        this.getCateList();
        this.addCateDialogVisible = false;
      })
    },
    async deleteCategoryById(row) {
      this.$confirm('此操作将永久删除该分类参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`categories/${row.cat_id}`);
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
        this.$message.success(res.meta.msg);
        this.getCateList();
      }).catch(() => {
        /* this.$message({
          type: 'info',
          message: '已取消删除'
        });  */         
      });
    },
    async getCateById(id) {
      const { data: res } = await this.$http.get(`categories/${id}`);
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.editCateFormData = res.data;
    },
    showEditCateDialog(row) {
      this.getCateById(row.cat_id);
      this.editCateDialogVisible = true;
    },
    async submitEditCate() {
      const { data: res } = await this.$http.put(`categories/${this.editCateFormData.cat_id}`, { cat_name: this.editCateFormData.cat_name });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.$message.success(res.meta.msg);
      this.editCateDialogVisible = false;
      this.getCateList();
    }
  }
}
</script>

<style>
  .zk-table {
    font-size: 14px !important;
  }
  .cateListClass {
    margin-top: 15px;
  }
</style>