<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddGoodsPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table
        :data="goodsList"
        border
        stripe>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column label="商品名称" prop="goods_name">
        </el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="105px">
        </el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="80px">
        </el-table-column>
        <el-table-column label="创建时间" width="180px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-tooltip 
              effect="dark" 
              content="编辑商品" 
              placement="top" 
              :enterable="false">
              <el-button 
                type="primary" 
                size="mini" 
                icon="el-icon-edit"
                round>
              </el-button>
            </el-tooltip>
            <el-tooltip 
              effect="dark" 
              content="删除商品" 
              placement="top" 
              :enterable="false">
              <el-button 
                type="danger" 
                size="mini" 
                icon="el-icon-delete"
                round 
                @click="deleteGoodById(scope.row.goods_id)">
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
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getGoodsList();
  },
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodsList: [],
      total: 0
    }
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.goodsList = res.data.goods;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    handleCurrentChange(newPagenum) {
      this.queryInfo.pagenum = newPagenum;
      this.getGoodsList();
    },
    deleteGoodById(id) {
      this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
          const { data: res } = await this.$http.delete('goods/' + id);
          if (res.meta.status !== 200) return this.$message.error("删除失败");
          this.$message.success("删除成功");
          this.getGoodsList();
      }).catch(() => {
        /* this.$message({
          type: 'info',
          message: '已取消删除'
        }); */          
      });
    },
    goAddGoodsPage() {
      this.$router.push('/goods/add');
    }
  }
}
</script>

<style>

</style>