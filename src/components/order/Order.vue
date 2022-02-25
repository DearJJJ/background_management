<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 搜索栏 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable>
            <el-button slot="append" icon="el-icon-search" @click="getOrderDataListByParam"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="toAddGoodPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格区 -->
      <el-table
        :data="orderList"
        border
        stripe>
        <el-table-column
          type="index">
        </el-table-column>
        <el-table-column
          label="订单编号"
          prop="order_number">
        </el-table-column>
        <el-table-column
          label="订单价格"
          prop="order_price">
        </el-table-column>
        <el-table-column
          label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="是否发货"
          prop="is_send"
          width="60px"
          align="center">
        </el-table-column>
        <el-table-column
          label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="130px">
          <template slot-scope="scope">
            <el-tooltip effect="dark" content="修改地址" placement="top" :enterable="false">
              <el-button 
                type="primary" 
                size="mini" 
                icon="el-icon-edit"
                @click="showEditAddressDialog"
                round>
              </el-button>
            </el-tooltip>
            <el-tooltip effect="dark" content="物流信息" placement="top" :enterable="false">
              <el-button
                type="success" 
                size="mini" 
                icon="el-icon-location"
                round
                @click="showLogisticsProcessDialog">
              </el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background>
      </el-pagination>

      <!-- 修改地址对话框 -->
      <el-dialog
        title="提示"
        :visible.sync="editAddressDialogVisible"
        width="40%"
        @close="resetEditAddressForm">
        <el-form 
          ref="editAddressFormRef" 
          :model="addressForm" 
          label-width="100px"
          :rules="addressFormRules">
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              :options="cityData"
              v-model="addressForm.address1">
            </el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editAddressDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="submitEditAddress">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 物流进度对话框 -->
      <el-dialog
        title="物流进度"
        :visible.sync="logisticsProcessDialogVisible"
        width="40%">
        <el-timeline :reverse="true">
          <el-timeline-item
            v-for="(activity, index) in logisticsList"
            :key="index"
            :timestamp="activity.time">
            {{ activity.context }}
          </el-timeline-item>
        </el-timeline>
        <span slot="footer" class="dialog-footer">
          <el-button @click="logisticsProcessDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="logisticsProcessDialogVisible = false">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from '@/components/order/citydata.js'
import logisticsData from '@/components/order/logisticsData.js'

export default {
  created() {
    this.getOrderDataList();
  },
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      orderList: [],
      total: 0,
      editAddressDialogVisible: false,
      logisticsProcessDialogVisible: false,
      addressForm: {
        address1: '',
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择市区县', trigger: 'blur' },
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      logisticsList: []
    }
  },
  methods: {
    async getOrderDataList() {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.total = res.data.total;
      this.orderList = res.data.goods;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderDataList();
    },
    handleCurrentChange(newPagenum) {
      this.queryInfo.pagenum = newPagenum;
      this.getOrderDataList();
    },
    showEditAddressDialog() {
      this.editAddressDialogVisible = true;
    },
    resetEditAddressForm() {
      this.$refs.editAddressFormRef.resetFields();
    },
    getLogisticsMessage() {
      /* const { data: res } = await this.$http.get(`/kuaidi/1106975712662`);
      console.log(res);
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.logisticsList = res.data; */
      this.logisticsList = logisticsData.data;
    },
    showLogisticsProcessDialog() {
      this.getLogisticsMessage();
      this.logisticsProcessDialogVisible = true;
    },
    getOrderDataListByParam() {
      this.$message.error('功能未开发！');
    },
    submitEditAddress() {
      this.$message.error('功能未开发！');
    },
    toAddGoodPage() {
      this.$router.push('/goods/add');
    }
  }
}
</script>

<style lang="less" scoped>
  .el-cascader {
    width: 100%;
  }
  
</style>