<template>
  <div>
    <div style="margin-top: 10px;display: flex;justify-content: center">
      <el-input
        v-model="keywords"
        placeholder="通过用户名搜索用户..."
        prefix-icon="el-icon-search"
        style="width: 400px;margin-right: 10px"
        @keydown.enter.native="doSearch"
      ></el-input>
      <el-button icon="el-icon-search" type="primary" @click="doSearch">搜索</el-button>
    </div>
    <div style="margin-top: 10px;display: flex;justify-content: left">
      <el-button type="primary" icon="el-icon-circle-plus-outline" @click="addRole">添加角色</el-button>
      <el-button type="primary" @click="addRoleMenus">分配菜单</el-button>
    </div>
    <div class="hr-container">
      <el-table
        :data="userDtaa"
        stripe
        style="width: 100%"
        @row-click="editRole"
        @selection-change="selectedChange"
      >
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="rid" label="角色编号" width="600"></el-table-column>
        <el-table-column prop="rname" label="角色名称"></el-table-column>
      </el-table>
    </div>

    <el-drawer
      title="新增角色"
      :visible.sync="drawer"
      :rules="rules"
      :with-header="false"
      :modal="false"
      ref="drawer"
    >
      <div style="margin-top:50px;">
        <el-form
          :model="roleForm"
          :rules="rules"
          ref="roleForm"
          :label-position="labelPosition"
          label-width="100px"
          style="margin-right:40px"
          action
        >
          <el-form-item v-if="roleForm.id==''" label="角色编码：" prop="rid">
            <el-input v-model="roleForm.rid"></el-input>
          </el-form-item>
          <el-form-item v-else label="角色编码：">
            <el-input v-model="roleForm.rid" readonly></el-input>
          </el-form-item>
          <el-form-item label="角色名称：" prop="rname">
            <el-input v-model="roleForm.rname"></el-input>
          </el-form-item>
        </el-form>

        <div style="margin-top: 10px;display: flex;justify-content: center">
          <el-button @click="cancelForm">取 消</el-button>
          <el-button
            v-if="roleForm.id==''"
            type="primary"
            @click="submitForm('roleForm','save')"
            :loading="loading"
          >{{ loading ? '提交中 ...' : '立即创建' }}</el-button>
          <el-button v-else type="primary" @click="submitForm('roleForm','edit')">修改</el-button>
        </div>
      </div>
    </el-drawer>

    <el-drawer
      title="新增菜单"
      :visible.sync="menusdrawer"
      :with-header="false"
      :modal="false"
      ref="menusdrawer"
    >
      <div style="margin-top:50px;">
        <div style="margin-top: 10px;display: flex;justify-content: center">
          <el-button @click="cancelForm">取 消</el-button>
          <el-button
            type="primary"
            @click="saveRoleMenus()"
            :loading="loading"
          >{{ loading ? '提交中 ...' : '保存' }}</el-button>
        </div>

        <div
          style="max-height: 700px;min-height: 500px;width:100%;position: absolute;overflow-y: auto;margin: auto;"
          v-loading="!showMenu"
        >
          <el-tree
            :data="menutreedata"
            show-checkbox
            default-expand-all
            node-key="id"
            ref="tree"
            highlight-current
            :props="defaultProps"
          ></el-tree>
        </div>
      </div>
    </el-drawer>
  </div>
</template>

<script>
export default {
  name: "SysRole",
  data() {
    var checkPhone = (rule, value, callback) => {
      const phoneReg = /^1[3|4|5|6|7|8][0-9]{9}$/;
      if (!value) {
        return callback(new Error("电话号码不能为空"));
      }
      setTimeout(() => {
        if (!Number.isInteger(+value)) {
          callback(new Error("请输入数字值"));
        } else {
          if (phoneReg.test(value)) {
            callback();
          } else {
            callback(new Error("电话号码格式不正确"));
          }
        }
      }, 100);
    };
    return {
      drawer: false, //抽屉
      menusdrawer: false,
      loading: false,

      roleForm: { id: "", rid: "", rname: "" }, //表单对象

      labelPosition: "right",
      userForm: {
        uid: "",
        username: "",
        password: "",
        name: "",
        phone: "",
        telephone: "",
        address: "",
        remark: "",
        enabled: true
      },
      //表单验证数据
      rules: {
        rid: [
          { required: true, message: "请输入角色编码", trigger: "blur" },
          { min: 3, max: 18, message: "长度在 3 到 18 个字符", trigger: "blur" }
        ],
        rname: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          { min: 1, max: 10, message: "长度在 1 到 10 个字符", trigger: "blur" }
        ]
      },

      keywords: "",
      userDtaa: [],
      selectedRoles: [],
      allroles: [],
      selectedRows: [],

      showMenu: false,
      menutreedata: [],
      defaultProps: {
        children: "children",
        label: "name"
      },
      roleMenus: []
    };
  },

  mounted() {
    this.initUser();
  },
  methods: {
    selectedChange(selection) {
      this.selectedRows.splice(0);
      for (let i in selection) {
        this.selectedRows.push(selection[i]);
      }
    },
    deleteHr(hr) {
      this.$confirm("此操作将永久删除【" + hr.name + "】, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.deleteRequest("/system/user/" + hr.id).then(resp => {
            if (resp) {
              this.initUser();
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    doSearch() {
      this.initUser();
    },
    hidePop(hr) {
      let roles = [];
      Object.assign(roles, hr.roles);
      let flag = false;
      if (roles.length != this.selectedRoles.length) {
        flag = true;
      } else {
        for (let i = 0; i < roles.length; i++) {
          let role = roles[i];
          for (let j = 0; j < this.selectedRoles.length; j++) {
            let sr = this.selectedRoles[j];
            if (role.id == sr) {
              roles.splice(i, 1);
              i--;
              break;
            }
          }
        }
        if (roles.length != 0) {
          flag = true;
        }
      }
      if (flag) {
        let url = "/system/hr/role?hrid=" + hr.id;
        this.selectedRoles.forEach(sr => {
          url += "&rids=" + sr;
        });
        this.putRequest(url).then(resp => {
          if (resp) {
            this.initUser();
          }
        });
      }
    },
    getMenuTreeData() {
      this.getRequest("/system/menu/listmenu").then(resp => {
        if (resp) {
          // console.log(resp);
          this.menutreedata = resp;
          this.getMenusByRole();
        }
      });
    },
    getMenusByRole() {
      let than = this;
      than.$refs.tree.setCheckedKeys([]);
      let rid = this.roleForm.rid;
      let menus = [];
      than.getRequest("/system/menu/getmenubyrid?rid=" + rid).then(resp => {
        if (resp) {
          menus.push(resp);
          for (var i = 0; i < menus.length; i++) {
            than.$refs.tree.setCheckedKeys(menus[i]);
          }
          than.showMenu = true;
        }
      });
    },
    saveRoleMenus(){
      let than = this;
      let arr = than.$refs.tree.getCheckedKeys();
      let menus = [];
      for(let i=0;i<arr.length;i++){
        if(arr[i]=='1' || arr[i]=='2' || arr[i]=='3' || arr[i]=='4' || arr[i]=='5' || arr[i]=='6' || arr[i]=='7' )
        continue;
        menus.push(arr[i]);

      }
      console.log(menus);
       than.getRequest("/system/menu/saverolemenus?menus="+menus+"&rid="+than.roleForm.rid).then(resp => {
        if (resp) {
          console.log(resp);
        }
      });
    },
    addRoleMenus() {
      // this.roleForm = { id: "", rid: "", rname: "" }; //清空表单对象
      let than = this;
      if (than.selectedRows.length != 1) {
        this.$message({
          message: "请选择一个角色",
          type: "warning"
        });
        return;
      }
      than.roleForm = { id: than.selectedRows[0].id, rid: than.selectedRows[0].rid, rname: than.selectedRows[0].rname };
      than.menusdrawer = true;
      than.getMenuTreeData();
    },
    enabledChange(user) {
      this.putRequest("/system/user/", user).then(resp => {
        if (resp) {
          this.initUser();
        }
      });
    },
    initAllRoles() {
      this.getRequest("/system/hr/roles").then(resp => {
        if (resp) {
          this.allroles = resp;
        }
      });
    },
    initUser() {
      this.getRequest("/system/role/listrole").then(resp => {
        if (resp) {
          console.log(resp);
          this.userDtaa = resp;
        }
      });
    },
    addRole() {
      this.showMenu = false;
      this.roleForm = { id: "", rid: "", rname: "" }; //清空表单对象
      this.drawer = true;
    },
    editRole(row, column, event) {
      this.roleForm = { id: row.id, rid: row.rid, rname: row.rname };
      this.drawer = true;
    },
    handleClose(done) {
      if (this.loading) {
        return;
      }
      this.$confirm("确定要提交表单吗？")
        .then(_ => {
          this.loading = true;
          this.timer = setTimeout(() => {
            done();
            // 动画关闭需要一定的时间
            setTimeout(() => {
              this.loading = false;
            }, 400);
          }, 2000);
        })
        .catch(_ => {});
    },
    cancelForm() {
      this.loading = false;
      this.drawer = false;
      clearTimeout(this.timer);
    },

    submitForm(formName, type) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.Loading = true;
          this.postRequest("/system/role/" + type, this.roleForm).then(resp => {
            if (resp) {
              this.Loading = false;
              this.drawer = false;
              this.initUser();
            }
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    }
  },
  created() {}
};
</script>

<style>
.userinfo-container div {
  font-size: 12px;
  color: #409eff;
}

.userinfo-container {
  margin-top: 20px;
}

.img-container {
  width: 100%;
  display: flex;
  justify-content: center;
}

.userface-img {
  width: 72px;
  height: 72px;
  border-radius: 72px;
}

.hr-container {
  margin-top: 10px;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.hr-card {
  width: 350px;
  margin-bottom: 20px;
}
</style>