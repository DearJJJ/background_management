<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 信息提示 -->
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false">
      </el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200" :active="+activeIndex" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- Form表单 -->
      <el-form 
        :model="addGoodForm" 
        :rules="addGoodFormRules" 
        label-position="top"
        ref="addGoodFormRef">
        <!-- Tab栏 -->
        <el-tabs :tab-position="'left'" v-model="activeIndex" @tab-click="handleTabClick" :before-leave="handleLeave">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addGoodForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addGoodForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addGoodForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addGoodForm.goods_cat"
                :options="cateList"
                :props="cascaderProps"
                @change="handleCascaderChange">
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item 
              v-for="item in manyParamsList" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  :label="attrVal" 
                  v-for="(attrVal, i) in item.attr_vals" 
                  :key="i">
                </el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              v-for="item in onlyParamsList"
              :key="item.attr_id"
              :label="item.attr_name">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :headers="headersObj"
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemovePicture"
              list-type="picture"
              :on-success="handUploadSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor
              v-model="addGoodForm.goods_introduce">
            </quill-editor>
            <el-button type="primary" @click="addGood">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 预览图片对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="previewImgDialogVisible"
      width="40%">
      <img :src="previewImgURL" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  created() {
    this.getCateList();
  },
  data() {
    return {
      headersObj: { Authorization: window.sessionStorage.getItem('token') },
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      activeIndex: '0',
      addGoodForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addGoodFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' },
        ]
      },
      cateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
      },
      // 动态参数列表
      manyParamsList: [],
      // 静态属性列表
      onlyParamsList: [],
      previewImgURL: '',
      previewImgDialogVisible: false
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories');
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.cateList = res.data;
    },
    handleCascaderChange() {
      if (this.addGoodForm.goods_cat.length < 3) return this.addGoodForm.goods_cat = [];
    },
    handleLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addGoodForm.goods_cat.length === 0) {
        this.$message.error('请先选择商品分类！')
        return false;
      }
    },
    async getManyParamsByCateId() {
      const { data: res } = await this.$http.get(`categories/${this.selectedThirdCateId}/attributes`, { params: { sel: 'many' } } );
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals.length === 0 ? null : item.attr_vals.split(' ');
      })
      this.manyParamsList = res.data;
    },
    async getOnlyParamsByCateId() {
      const { data: res } = await this.$http.get(`categories/${this.selectedThirdCateId}/attributes`, { params: { sel: 'only' } } );
      /* if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals.length === 0 ? null : item.attr_vals.split(' ');
      }) */
      this.onlyParamsList = res.data;
    },
    handleTabClick() {
      if (this.activeIndex === '1') {
        return this.getManyParamsByCateId();
      }
      if (this.activeIndex === '2') {
        return this.getOnlyParamsByCateId();
      }
    },
    handlePreview(file) {
      this.previewImgURL = file.response.data.url;
      this.previewImgDialogVisible = true;
    },
    handleRemovePicture(file) {
      this.addGoodForm.pics = this.addGoodForm.pics.filter(item => item.pic !== file.response.data.tmp_path);
    },
    handUploadSuccess(response) {
      const picObj = { pic: response.data.tmp_path }
      this.addGoodForm.pics.push(picObj);
    },
    // 处理静态属性
    handleOnlyParams(formData) {
      formData.forEach(item => {
        const paramObj = { attr_id: item.attr_id, attr_value: item.attr_vals }
        this.addGoodForm.attrs.push(paramObj);
      })
    },
    // 处理动态参数
    handleManyParams(formData) {
      formData.forEach(item => {
        const paramObj = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
        this.addGoodForm.attrs.push(paramObj);
      })
    },
    addGood() {
      this.$refs.addGoodFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的表单项');
        const cloneAddGoodForm = _.cloneDeep(this.addGoodForm);
        cloneAddGoodForm.goods_cat = cloneAddGoodForm.goods_cat.join(',');
        this.handleOnlyParams(this.onlyParamsList);
        this.handleManyParams(this.manyParamsList);
        cloneAddGoodForm.attrs = this.addGoodForm.attrs;
        const { data: res } = await this.$http.post('goods', cloneAddGoodForm);
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg);
        this.$message.success(res.meta.msg);
        this.$router.push('/goods');
      })
    }
  },
  computed: {
    selectedThirdCateId() {
      if (this.addGoodForm.goods_cat.length !== 3) return null;
      return this.addGoodForm.goods_cat[2];
    }
  }
}
</script>

<style lang="less" scoped>
  .el-checkbox {
    margin: 0 5px 0 0 !important;
  }
  .previewImg {
    width: 100%;
  }
  /deep/ .ql-editor {
    min-height: 300px;
  }
  .el-button {
    margin-top: 10px;
  }
</style>