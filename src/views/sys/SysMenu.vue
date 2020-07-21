<template>
  <div>
    <div style="margin-top: 10px;display: flex;justify-content: left">
      <!-- <el-tree
        :data="menutreedata"
        show-checkbox
        default-expand-all
        node-key="id"
        ref="tree"
        highlight-current
        :props="defaultProps"
      ></el-tree>-->
    </div>

    <div>
      <el-table
        :data="menutreedata"
        style="width: 100%;margin-bottom: 20px;"
        row-key="id"
        border
        stripe
        default-expand-all
        highlight-current-row
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
      >
        <el-table-column prop="url" label="请求地址"></el-table-column>
        <el-table-column prop="path" label="菜单地址" width="500"></el-table-column>
        <el-table-column prop="component" label="组件名" width="250"></el-table-column>
        <el-table-column prop="name" label="菜单名称" width="300"></el-table-column>
        <el-table-column prop="iconcls" label="图标样式" width="200"></el-table-column>
        <el-table-column prop="requireAuth" label="认证请求" width="150"></el-table-column>
        <el-table-column prop="pid" label="上级菜单" width="150"></el-table-column>
        <el-table-column prop="enabled" label="已启用" width="120"></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" @click="handleAdd(scope.$index, scope.row)">添加</el-button>
            <el-button size="mini" type="success" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <el-dialog :title="isAdd?'添加菜单':'编辑菜单'" :visible.sync="editMenuDialog">
      <el-form :model="menuForm">
        <el-form-item label="请求地址" :label-width="formLabelWidth">
          <el-input v-model="menuForm.url" ></el-input>
        </el-form-item>
        <el-form-item label="菜单地址" :label-width="formLabelWidth">
          <el-input v-model="menuForm.path" ></el-input>
        </el-form-item>
        <el-form-item label="组件名称" :label-width="formLabelWidth">
          <el-input v-model="menuForm.component" ></el-input>
        </el-form-item>
        <el-form-item label="菜单名称" :label-width="formLabelWidth">
          <el-input v-model="menuForm.name" ></el-input>
        </el-form-item>
        <el-form-item label="菜单样式" :label-width="formLabelWidth">
          <el-input v-model="menuForm.iconcls" ></el-input>
        </el-form-item>
        <!-- <el-form-item label="认证请求" :label-width="formLabelWidth">
          <el-input v-model="menuForm.requireAuth" ></el-input>
        </el-form-item> -->
        <el-form-item v-if="!isAdd" label="上级菜单" :label-width="formLabelWidth">
          <el-input v-model="menuForm.pid"></el-input>
        </el-form-item>
        <el-form-item v-else label="上级菜单" :label-width="formLabelWidth">
          <el-input v-model="menuForm.pid" readonly></el-input>
        </el-form-item>
        <!-- <el-form-item label="已启用" :label-width="formLabelWidth">
          <el-input v-model="menuForm.enabled" ></el-input>
        </el-form-item> -->
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveMenu">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "SysMenu",
  props: ["keys"],
  data() {
    return {
      drawer: false, //抽屉
      loading: false,

      editMenuDialog: false,
      isAdd: true,
      formLabelWidth: "120px",
      menuForm: {
        id: "",
        url: "",
        path: "",
        component: "",
        name: "",
        iconcls: "",
        requireAuth: "",
        pid: "",
        enabled: true,
      },
      menutreedata: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    initMenuTreeData() {
      this.getRequest("/system/menu/listmenu").then(resp => {
        if (resp) {
          console.log(resp);
          this.menutreedata = resp;
        }
      });
    },
    handleAdd(index, row) {
      this.isAdd = true;
      console.log(index, row);
      this.menuForm = {
        id: "",
        url: "",
        path: "",
        component: "",
        name: "",
        iconcls: "",
        requireAuth: 1,
        pid: row.id,
        enabled: 1
      };
      this.editMenuDialog = true;
    },
    handleEdit(index, row) {
      this.isAdd = false;
      this.menuForm = {
        id: row.id,
        url: row.url,
        path: row.path,
        component: row.component,
        name: row.name,
        iconcls: row.iconcls,
        requireAuth: row.requireAuth,
        pid: row.pid,
        enabled: row.enabled
      };
      console.log(index, row);
      this.editMenuDialog = true;
    },
    handleDelete(index, row) {
      let mid = row.id;
      console.log(index, row);
    },
    saveMenu() {
      let than = this;
      let type = "";
      if (than.isAdd) {
        type = "add";
      } else {
        type = "tdit";
      }
      console.log(than.menuForm);
      let ajax = [];
      ajax.push(than.postRequest("/system/menu/"+type, than.menuForm ));
      Promise.all(ajax)
        .then(function(resp) {
          console.log(resp);
          than.editMenuDialog = false;
          than.initMenuTreeData();
        })
        .catch(function(e) {
          than.$message(e);
          than.editMenuDialog = true;
        });

      // this.getRequest("/system/menu/"+type,than.menuForm).then(resp => {
      //   if (resp) {
      //     console.log(resp);
      //     than.editMenuDialog = false;
      //     than.initMenuTreeData();
      //   }else{
      //     than.editMenuDialog =  true;
      //   }
      // });
    },
    //     submitForm(formName,type) {
    //   let than = this;
    //   console.log(than.userForm)
    //   this.$refs[formName].validate(valid => {
    //     if (valid) {
    //       let ajax = [];
    //       ajax.push(than.postRequest("/system/user/"+type, than.userForm));
    //       if(than.oldselectedRoles.toString()!=than.selectedRoles.toString()){
    //         ajax.push(than.postRequest("/system/user/role?uid="+than.userForm.uid+"&rids="+than.selectedRoles));
    //       }
    //       Promise.all(ajax).then(function(resp){
    //         console.log(resp);
    //         than.drawer = false;
    //         than.initUser();
    //       }).catch(function(e){
    //         than.$message(e);
    //         than.drawer = true;
    //       });
    //     } else {
    //       console.log("error submit!!");
    //       return false;
    //     }
    //   });
    // }
  },
  created() {
    this.initMenuTreeData();
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
</style>