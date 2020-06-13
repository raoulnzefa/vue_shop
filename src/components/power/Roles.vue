<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限区域 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限区域 -->
              <el-col :span="19">
                <!-- 通过 for 循环嵌套渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template v-slot="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showRoleName(scope.row.id)"
            >编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
            >删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色的对话框区域 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="50%"
      @close="addRoleDialogClose"
    >
      <!-- 内容主体区域 -->
      <span>
        <el-form
          :model="addRoleForm"
          :rules="editFormRules"
          ref="addRoleFormRef"
          label-width="80px"
        >
          <el-form-item label="用户名称" prop="roleName">
            <el-input v-model="addRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="用户描述" prop="roleDesc">
            <el-input v-model="addRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色信息的对话框 -->
    <el-dialog
      title="编辑信息"
      :visible.sync="editRoleNameVisible"
      width="50%"
      @close="editRoleDialogClose"
    >
      <span>
        <el-form :model="editRole" :rules="editFormRules" ref="editRoleRef" label-width="80px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRole.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRole.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleNameVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog
      @close="setRightDialogClose"
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
    >
      <!-- 树形控件 -->
      <el-tree
        show-checkbox
        node-key="id"
        default-expand-all
        :data="rightsList"
        :props="treeProps"
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "Roles",
  data() {
    return {
      // 所有角色列表数据
      rolesList: [],
      // 编辑信息弹出局域栏的显示与隐藏
      editRoleNameVisible: false,
      // 修改信息的验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: "请输入新的用户名称", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "长度在 3 到 10 个字符",
            trigger: "blur"
          }
        ],
        roleDesc: [
          { required: true, message: "请输入新的用户描述", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "长度在 3 到 10 个字符",
            trigger: "blur"
          }
        ]
      },
      // 查询到的角色信息
      editRole: {},
      // 添加角色弹出框的显示和隐藏
      addRoleDialogVisible: false,
      // 添加角色的表单数据
      addRoleForm: {
        roleName: "",
        roleDesc: ""
      },
      // 分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        children: "children",
        label: "authName"
      },
      // 默认选中的节点 Id 值数组
      defKeys: [],
      // 当前角色的 id
      roleId: ""
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    // 获取所有角色列表信息
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.rolesList = res.data;
    },
    // 弹出编辑信息框
    async showRoleName(id) {
      const { data: res } = await this.$http.get(`roles/${id}`);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.editRole = res.data;
      this.editRoleNameVisible = true;
    },
    // 提交编辑信息
    editRoleInfo() {
      this.$refs.editRoleRef.validate(async valid => {
        if (!valid) {
          return;
        }
        // 发起修改请求
        const { data: res } = await this.$http.put(
          `roles/${this.editRole.roleId}`,
          {
            roleName: this.editRole.roleName,
            roleDesc: this.editRole.roleDesc
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg);
        }
        this.editRoleNameVisible = false;
        this.getRolesList();
        this.$message.success("修改成功");
      });
    },
    // 删除用户
    async removeRoleById(id) {
      // 首先应弹出弹窗询问是否删除
      const confirmResult = await this.$confirm(
        "此操作将永久删除该角色, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      const { data: res } = await this.$http.delete(`roles/${id}`);
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.$message.success(res.meta.msg);
      this.getRolesList();
    },
    // 监听编辑弹出框的关闭事件
    editRoleDialogClose() {
      this.$refs.editRoleRef.resetFields();
    },
    // 提交添加角色信息
    addRoleInfo() {
      this.$refs.addRoleFormRef.validate(async valid => {
        if (!valid) {
          return;
        }
        // 可以发起添加的网络请求
        const { data: res } = await this.$http.post("roles", this.addRoleForm);
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg);
        }
        // 隐藏添加角色的对话框
        this.addRoleDialogVisible = false;
        this.getRolesList();
        this.$message.success(res.meta.msg);
      });
    },
    // 添加角色弹出栏关闭时清除验证状态
    addRoleDialogClose() {
      this.$refs.addRoleFormRef.resetFields();
    },
    // 移除角色权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      role.children = res.data;
      this.$message.success(res.meta.msg);
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id;
      // 获取所有权限的数据
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      // 把获取到的权限数据保存到 data 中
      this.rightsList = res.data;
      // 递归获取三级节点的 id
      this.getLeafKeys(role, this.defKeys);
      this.setRightDialogVisible = true;
    },
    // 通过递归的形式获取角色下所有三级权限的 id 并保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node节点不包含 children属性，说明是三级节点
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach(item => this.getLeafKeys(item, arr));
    },
    // 在关闭弹窗后清空 defKeys数组，因为用的是 push
    setRightDialogClose() {
      this.defKeys = [];
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ];
      const idStr = keys.join(",");
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        {
          rids: idStr
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg);
      }
      this.$message.success(res.meta.msg);
      this.getRolesList();
      this.setRightDialogVisible = false;
    }
  }
};
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>