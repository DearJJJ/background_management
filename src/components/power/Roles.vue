<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table
        :data="roleList"
        border
        stripe>
        <el-table-column
          type="expand">
          <template slot-scope="scope">
            <el-row 
              v-for="(item1, i1) in scope.row.children" 
              :key="item1.id"
              :class="['bdbot', i1 === 0 ? 'bdtop' :'', 'vcenter']"
              align="center">
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)">
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children" 
                  :key="item2.id"
                  :class="[i2 === 0 ? '' :'bdtop', 'vcenter']">
                  <el-col :span="6">
                    <el-tag 
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)">
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag 
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)">
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column
          type="index"
          label="#">
        </el-table-column>
        <el-table-column
          prop="roleName"
          label="角色名称">
        </el-table-column>
        <el-table-column
          prop="roleDesc"
          label="角色描述">
        </el-table-column>
        <el-table-column
          label="操作"
          width="300px">
            <template slot-scope="scope">
              <el-button icon="el-icon-edit" size="mini" type="primary" round>编辑</el-button>
              <el-button icon="el-icon-delete" size="mini" type="danger" round>删除</el-button>
              <el-button 
                icon="el-icon-setting" 
                size="mini" 
                type="warning" 
                round
                @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 设置权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="rightDialogVisible"
      width="40%">
      <el-tree 
        :data="rightTree" 
        :props="rightTreeProps"
        ref="rightTreeRef"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="checkedKeys">
      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created() {
    this.getRoleList();
  },
  data() {
    return {
      roleList: [],
      rightTree: [],
      rightDialogVisible: false,
      rightTreeProps: {
        children: 'children',
        label: 'authName'
      },
      checkedKeys: [],
      rightTreeRoleId: ''
    }
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles');
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.roleList = res.data;
    },
    async removeRightById(role, rightId) {
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`);
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
        this.$message.success(res.meta.msg);
        role.children = res.data;
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    async showSetRightDialog(rootRole) {
      this.checkedKeys = [];
      this.rightTreeRoleId = rootRole.id;
      const { data: res } = await this.$http.get('rights/tree');
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.getLeafKeys(rootRole, this.checkedKeys);
      this.rightTree = res.data;
      this.rightDialogVisible = true;
    },
    getLeafKeys(node, arr) {
      if (!node.children) return arr.push(node.id);
      node.children.forEach((item) => {
        this.getLeafKeys(item, arr);
      })
    },
    async allotRights() {
      const keyArr = [...this.$refs.rightTreeRef.getCheckedKeys(), ...this.$refs.rightTreeRef.getHalfCheckedKeys()];
      const keyArrStr = keyArr.join(',');
      const { data: res } = await this.$http.post(`roles/${this.rightTreeRoleId}/rights`, { rids: keyArrStr });
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.$message.success(res.meta.msg);
      this.rightDialogVisible = false;
      this.getRoleList();
    }
  }
}
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }
  .bdtop {
    border-top: 1px solid #eee;
  }
  .bdbot {
    border-bottom: 1px solid #eee;
  }
  .vcenter {
    display: flex;
    align-items: center;
  }
</style>