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
      <el-button type="primary" icon="el-icon-circle-plus-outline" @click="addUser">添加用户</el-button>
      <!-- <el-button type="primary" icon="el-icon-edit" @click="addUser">编辑</el-button> -->
      <el-button type="primary" icon="el-icon-delete" v-if="this.$store.state.currentHr.username=='admin'" @click="deleteUser">删除</el-button>
    </div>
    <div class="hr-container">
      <el-table
        :data="userDtaa"
        stripe
        style="width: 100%;max-height:700px;"
        @row-click="editUser"
        @selection-change="selectedChange"
      >
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="uid" label="工号" width="180"></el-table-column>
        <el-table-column prop="username" label="账号" width="180"></el-table-column>
        <el-table-column prop="name" label="姓名" width="180"></el-table-column>
        <el-table-column prop="phone" label="手机" width="180"></el-table-column>
        <el-table-column prop="telephone" label="电话" width="180"></el-table-column>
        <el-table-column prop="address" label="地址" width="250"></el-table-column>
        <el-table-column prop="remark" label="备注"></el-table-column>
        <el-table-column prop="enabled" label="启用" width="150">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.enabled"
              active-text
              @change="enabledChange(scope.row)"
              active-color="#13ce66"
              inactive-color="#ff4949"
              inactive-text
            ></el-switch>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <el-drawer
      title="新增用户"
      :visible.sync="drawer"
      :rules="rules"
      :with-header="false"
      :modal="false"
      ref="drawer"
    >
      <div style="margin-top:50px;">
        <el-form
          :model="userForm"
          :rules="rules"
          ref="userForm"
          :label-position="labelPosition"
          label-width="100px"
          style="margin-right:40px"
          action
        >
          <el-form-item v-if="userForm.id==''" label="工号：" prop="uid">
            <el-input v-model="userForm.uid"></el-input>
          </el-form-item>
          <el-form-item v-else label="工号：">
            <el-input v-model="userForm.uid" readonly></el-input>
          </el-form-item>
          <el-form-item v-if="userForm.id==''" label="账号：" prop="username">
            <el-input v-model="userForm.username"></el-input>
          </el-form-item>
          <el-form-item v-else label="账号：" >
            <el-input v-model="userForm.username" readonly></el-input>
          </el-form-item>
          <el-form-item label="密码：" prop="password" v-if="userForm.id==''">
            <el-input v-model="userForm.password"></el-input>
          </el-form-item>
          <el-form-item label="姓名：" prop="name">
            <el-input v-model="userForm.name"></el-input>
          </el-form-item>
          <el-form-item label="手机：" prop="phone">
            <el-input v-model="userForm.phone"></el-input>
          </el-form-item>
          <el-form-item label="电话：" prop="telephone">
            <el-input v-model="userForm.telephone"></el-input>
          </el-form-item>
          <el-form-item label="地址：">
            <el-input v-model="userForm.address"></el-input>
          </el-form-item>
          <el-form-item label="备注：">
            <el-input v-model="userForm.remark"></el-input>
          </el-form-item>
          <el-form-item label="是否启用：">
            <el-switch
              v-model="userForm.enabled"
              active-text
              active-color="#13ce66"
              inactive-color="#ff4949"
              inactive-text
            ></el-switch>
          </el-form-item>
          <el-form-item label="所属角色：">
            <el-select v-model="selectedRoles" multiple  placeholder="请选择" style="width:100%">
              <el-option
                v-for="item in allroles"
                :key="item.rid"
                :label="item.rname"
                :value="item.rid"
              ></el-option>
            </el-select>
          </el-form-item>
          <!-- <div
            style="height:100px;min-height:100px;max-height:300px;border:1px solid rgb(220, 223, 230);margin:0 0 0 15px;padding: 5px;border-radius:3px;overflow-y:auto;"
          >
            <el-tag
              v-for="tag in selectedRoles"
              effect="Plain"
              class="role-tag"
              :key="tag.id"
              closable
            >{{tag.name}}</el-tag>
          </div> -->
        </el-form>
        <div style="margin-top: 10px;display: flex;justify-content: center">
          <el-button @click="cancelForm">取 消</el-button>
          <el-button v-if="userForm.id==''" type="primary" @click="submitForm('userForm','save')">立即创建</el-button>
          <el-button v-else type="primary" @click="submitForm('userForm','edit')">保存</el-button>
          <!-- <el-button
            type="primary"
            @click="$refs.drawer.closeDrawer()"
            :loading="loading"
          >{{ loading ? '提交中 ...' : '确 定' }}</el-button>-->
        </div>
      </div>
    </el-drawer>

    <!-- <el-dialog title="角色" :visible.sync="dialogTableVisible">
      <div>

      </div>
    </el-dialog>-->
  </div>
</template>

<script>
export default {
  name: "SysUser",
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
      loading: false,
      labelPosition: "right",
      dialogTableVisible: false,
      userForm: {
        id: "",
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
      // userRoles: [
      // ],
      //表单验证数据
      rules: {
        uid: [
          { required: true, message: "请输入工号", trigger: "blur" },
          { min: 2, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ],
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 10 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 6, max: 18, message: "长度在 6 到 18 个字符", trigger: "blur" }
        ],
        name: [
          { required: true, message: "请输入姓名", trigger: "blur" },
          { min: 2, max: 5, message: "长度在 2 到 5 个字符", trigger: "blur" }
        ],
        phone: [{ required: true, validator: checkPhone, trigger: "blur" }],
        telephone: [
          { min: 3, max: 5, message: "请输入正确的电话号码", trigger: "blur" }
        ],
        address: []
      },

      keywords: "",
      userDtaa: [],
      selectedRoles: [],
      oldselectedRoles: [],
      allroles: [],

      selectedRows:[],
      // userForm:{id:"",uid:"",username:"",password:"",name:"",phone:"",telephone:"",address:""},
    };
  },

  mounted() {
    this.initUser();
    this.initAllRoles();
  },
  methods: {
    selectedChange(	selection ){
      this.selectedRows.splice(0);
      for(let i=0; i<selection.length; i++ ){
          this.selectedRows.push(selection[i].uid);
      }
      console.log(this.selectedRows)
    },
    deleteUser() {
      let than = this;
      this.$confirm("此操作将永久删除选中用户, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.deleteRequest("/system/user/delete?uids=" + than.selectedRows ).then(resp => {
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
    enabledChange(user) {
      this.putRequest("/system/user/", user).then(resp => {
        if (resp) {
          this.initUser();
        }
      });
    },
    initAllRoles() {
        this.getRequest("/system/role/listrole").then(resp => {
        if (resp) {
          this.allroles = resp;
        }
      });
    },
    initUser() {
      this.getRequest("/system/user/?keywords=" + this.keywords).then(resp => {
        if (resp) {
          this.userDtaa = resp;
        }
      });
    },
    addUser() {
      this.selectedRoles.splice(0);
      this.userForm = {
        id: "",
        uid: "",
        username: "",
        password: "",
        name: "",
        phone: "",
        telephone: "",
        address: "",
        remark: "",
        enabled: true,
      };
      this.drawer = true;
    },
    editUser(row, column, event) {
      this.initselectRole(row.uid);
      this.userForm = {
        id: row.id,
        uid: row.uid,
        username: row.username,
        password: row.password,
        name: row.name,
        phone: row.phone,
        telephone: row.telephone,
        address: row.address,
        remark: row.remark,
        enabled: row.enabled,
      };
      this.drawer = true;
    },
    initselectRole(uid) {
      this.postRequest("/system/role/getrolesbyuid?uid="+uid).then(resp => {
            this.globalLoading = false;
            if (resp) {
              this.oldselectedRoles = resp;
              this.selectedRoles = resp;
            }
          });
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

    submitForm(formName,type) {
      let than = this;
      console.log(than.userForm)
      this.$refs[formName].validate(valid => {
        if (valid) {
          let ajax = [];
          ajax.push(than.postRequest("/system/user/"+type, than.userForm));
          if(than.oldselectedRoles.toString()!=than.selectedRoles.toString()){
            ajax.push(than.postRequest("/system/user/role?uid="+than.userForm.uid+"&rids="+than.selectedRoles));
          }
          Promise.all(ajax).then(function(resp){
            console.log(resp);
            than.drawer = false;
            than.initUser();
          }).catch(function(e){
            than.$message(e);
            than.drawer = true;
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    }
  }
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
.role-tag {
  margin: 5px 0 0 5px;
}
</style>