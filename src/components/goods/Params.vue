<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 警告区 -->
      <el-alert
        show-icon
        title="注意:只允许为第三级分类设置相关参数!"
        type="warning"
        :closable="false">
      </el-alert>
      <!-- 选择商品分类 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类: </span>
          <el-cascader
          v-model="selectedCateIds"
          :options="cateList"
          :props="cascaderProps"
          @change="handleCascaderChange">
        </el-cascader>
        </el-col>
      </el-row>
      <!-- Tab栏 -->
      <el-tabs v-model="activeName" @tab-click="handleClickTab">
        <!-- 动态参数栏 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button 
            type="primary" 
            size="mini" 
            :disabled="isBtnDisabled"
            @click="showAddCateParamsDialog">添加参数
          </el-button>
          <el-table
            :data="cateParamsList"
            border
            stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  v-for="(val, i) in scope.row.attr_vals"
                  :key="i"
                  @close="handleCloseTag(i, scope.row)">
                  {{ val }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputTagVisible"
                  v-model="scope.row.newTagInputValue"
                  ref="saveTagInputRef"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditParamDialog(scope.row.attr_id)">操作</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete"
                @click="deleteCateParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性栏 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button 
            type="primary" 
            size="mini" 
            :disabled="isBtnDisabled"
            @click="showAddCateParamsDialog">添加属性
          </el-button>
          <el-table
            :data="cateParamsList"
            border
            stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  v-for="(val, i) in scope.row.attr_vals"
                  :key="i"
                  @close="handleCloseTag(i, scope.row)">
                  {{ val }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputTagVisible"
                  v-model="scope.row.newTagInputValue"
                  ref="saveTagInputRef"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditParamDialog(scope.row.attr_id)">操作</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete"
                @click="deleteCateParam(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数/属性对话框 -->
    <el-dialog
      :title="'添加' + dialogText"
      :visible.sync="dialogVisible"
      width="40%"
      @close="addParamsDialogClosed">
      <el-form
        ref="addCateParamsFormRef" 
        :model="addCateParamFormData" 
        label-width="100px"
        :rules="addCateParamsFormRules">
        <el-form-item prop="attr_name" :label="dialogText + '名称：'">
          <el-input v-model="addCateParamFormData.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数/属性对话框 -->
    <el-dialog
      :title="'修改' + dialogText"
      :visible.sync="editDialogVisible"
      width="40%"
      @close="editParamsDialogClosed">
      <el-form
        ref="editCateParamsFormRef" 
        :model="editCateParamFormData" 
        label-width="100px"
        :rules="editCateParamsFormRules">
        <el-form-item prop="attr_name" :label="dialogText + '名称：'">
          <el-input v-model="editCateParamFormData.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="alterCateParam">确 定</el-button>
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
      cateList: [],
      cateParamsList: [],
      cascaderProps: { 
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
      },
      selectedCateIds: [],
      activeName: 'many',
      dialogVisible: false,
      editDialogVisible: false,
      addCateParamFormData: {
        attr_name: ''
      },
      editCateParamFormData: {
        attr_name: '',
        attr_id: ''
      },
      addCateParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数/属性名称', trigger: 'blur' }
        ]
      },
      editCateParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数/属性名称', trigger: 'blur' }
        ]
      },
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories');
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.cateList = res.data;
    },
    async getCateParamsList() {
      const { data: res } = await this.$http.get(`categories/${this.selectedThirdCateId}/attributes`, { params: { sel: this.activeName } });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : [];
        item.inputTagVisible = false;
        item.newTagInputValue = '';
      })
      this.cateParamsList = res.data;
    },
    async getCateParamById(id) {
      const { data: res } = await this.$http.get(`categories/${this.selectedThirdCateId}/attributes/${id}`, { params: { sel: this.activeName } });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.editCateParamFormData.attr_name = res.data.attr_name;
      this.editCateParamFormData.attr_id = res.data.attr_id;
    },
    handleCascaderChange() {
      if (this.selectedCateIds.length < 3) {
        this.selectedCateIds = [];
        this.cateParamsList = [];
        return;
      }
      this.getCateParamsList();
    },
    handleClickTab() {
      if (this.selectedCateIds.length < 3) return;
      this.getCateParamsList();
    },
    showAddCateParamsDialog() {
      this.dialogVisible = true;
    },
    showEditParamDialog(attrId) {
      this.getCateParamById(attrId);
      this.editDialogVisible = true;
    },
    alterCateParam() {
      this.$refs.editCateParamsFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.put(`categories/${this.selectedThirdCateId}/attributes/${this.editCateParamFormData.attr_id}`, {
          attr_name: this.editCateParamFormData.attr_name,
          attr_sel: this.activeName
        });
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
        this.$message.success(res.meta.msg);
        this.editDialogVisible = false;
        this.getCateParamsList();
      })
    },
    deleteCateParam(attrId) {
      this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
          const { data: res } = await this.$http.delete(`categories/${this.selectedThirdCateId}/attributes/${attrId}`);
          if (res.meta.status !== 200) return this.$message.error("删除失败");
          this.$message.success("删除成功");
          this.getCateParamsList();
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });          
      });
    },
    addParamsDialogClosed() {
      this.$refs.addCateParamsFormRef.resetFields();
    },
    editParamsDialogClosed() {
      this.$refs.editCateParamsFormRef.resetFields();
    },
    addParams() {
      this.$refs.addCateParamsFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post(`categories/${this.selectedThirdCateId}/attributes`, { 
          attr_name: this.addCateParamFormData.attr_name,
          attr_sel: this.activeName 
        })
        if (res.meta.status !== 201) this.$message.error(res.meta.msg);
        this.$message.success(res.meta.msg);
        this.getCateParamsList();
        this.dialogVisible = false;
      })
    },
    async saveAttrValues(row) {
      const { data: res } = await this.$http.put(`categories/${this.selectedThirdCateId}/attributes/${row.attr_id}`, { 
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      });
      if (res.meta.status !== 200) this.$message.error(res.meta.msg);
      this.$message.success(res.meta.msg);
    },
    handleInputConfirm(row) {
      if (row.newTagInputValue.trim().length === 0) {
        row.newTagInputValue = '';
        row.inputTagVisible = false;
        return;
      }
      row.attr_vals.push(row.newTagInputValue);
      row.newTagInputValue = '';
      row.inputTagVisible = false;
      this.saveAttrValues(row);
    },
    // 保存商品参数属性
    handleCloseTag(i, row) {
      row.attr_vals.splice(i, 1);
      this.saveAttrValues(row);
    },
    showInput(row) {
      row.inputTagVisible = true;
      this.$nextTick(() => {
        this.$refs.saveTagInputRef.$refs.input.focus();
      });
    }
  },
  computed: {
    isBtnDisabled() {
      return this.selectedCateIds.length < 3 ? true : false;
    },
    dialogText() {
      return this.activeName === 'many' ? '参数' : '属性';
    },
    selectedThirdCateId() {
      return this.selectedCateIds.slice(-1)[0];
    }
  }
}
</script>

<style lang="less" scoped>
  .cat_opt {
    margin: 15px 0;
  }
  .el-tag {
    margin: 10px;
  }
  .input-new-tag {
    width: 120px;
    margin: 10px;
  }
  .button-new-tag {
    margin: 10px;
  }
</style>