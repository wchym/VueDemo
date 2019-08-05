<template>
  <div>
    <el-form :inline="true" class="demo-form-inline">
      <el-button type="success" icon="el-icon-plus" plain @click="addUser()" style="float:left"></el-button>
      <el-button type="warning" icon="el-icon-delete" plain @click="deleteBatch()" style="float:left">批删</el-button>
      <el-form-item label="审批人">
        <el-input v-model="mydata.name" placeholder="审批人"></el-input>
      </el-form-item>
      <el-form-item label="出生日期">
        <el-input type="date" v-model="mydata.start" placeholder="开始日期"></el-input>
      </el-form-item>
      --
      <el-form-item label="">
        <el-input type="date" v-model="mydata.end" placeholder="结束日期"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit111">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      ref="multipleTable"
      :data="tableData"
      tooltip-effect="dark"
      style="width: 100%"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        prop="id"
        label="#"
        width="120">
        <!--<template slot-scope="scope">{{ scope.row.date }}</template>-->
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="120">
      </el-table-column>
      <el-table-column
        prop="sex"
        label="性别"
        width="120">
      </el-table-column>
      <el-table-column
        prop="birthday"
        label="生日"
        show-overflow-tooltip>
      </el-table-column>
      <el-table-column
        label="头像"
        show-overflow-tooltip>
        <template slot-scope="scope">
          <el-image
            style="width: 50px; height: 50px"
            :src=scope.row.imgaddr>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        show-overflow-tooltip>
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" plain @click="updateUser(scope.row)"></el-button>
          <el-button type="danger" icon="el-icon-delete" plain @click="deleteUser(scope.row.id)"></el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      background
      layout="total, prev, pager, next, sizes"
      :total="total"
      :page-sizes="[1, 2, 3, 4, 5]"
      :page-size="mydata.pageSize"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange">
    </el-pagination>

    <el-dialog title="用户信息" :visible.sync="dialogVisible">
      <el-form :model="form">
        <el-form-item label="#" :label-width="formLabelWidth">
          <el-input v-model="form.id" autocomplete="off" :readonly="true"></el-input>
        </el-form-item>
        <el-form-item label="用户姓名" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="用户性别" :label-width="formLabelWidth">
          <el-radio v-model="form.sex" label="男">男</el-radio>
          <el-radio v-model="form.sex" label="女">女</el-radio>
        </el-form-item>
        <el-form-item label="出生日期" :label-width="formLabelWidth">
          <el-input type="date" v-model="form.birthday" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="用户头像" :label-width="formLabelWidth">
          <el-upload
            class="avatar-uploader"
            action="http://localhost:10000/test/upload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="imageUrl" :src="imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="toclose">取 消</el-button>
        <el-button type="primary" @click="save">保 存</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    name: "UserList",
    data() {
      return {
        tableData: [],
        multipleSelection: [],
        total: 5,
        mydata: {
          page: 1,
          pageSize: 3,
          start: "",
          end: "",
          name: ""
        },
        form:{},
        formLabelWidth: '120px',
        dialogVisible: false,
        imageUrl:""
      }
    },
    mounted() {
      this.getList();
    },
    methods: {
      handleAvatarSuccess(res, file) {
        this.imageUrl = URL.createObjectURL(file.raw);
      },
      beforeAvatarUpload(file) {
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isLt2M;
      },
      deleteBatch(){
        let ids = "";
        // 这两种方法均可
        // this.multipleSelection.forEach((mod)=>{
        //   ids = ids + "," + mod.id;
        // })
        // ids = ids.substring(1);
        ids = this.multipleSelection.map(mod=>{
          return mod.id;
        })
        this.$axios.post("http://localhost:10000/test/deleteUserBatch?ids="+ids).then(respost => {
          if(respost.data.code == 200){
            if((this.total-1) % this.mydata.pageSize == 0){
              this.mydata.page=this.mydata.page-1;
            }
            this.getList();
            this.$message({
              message: '删除成功',
              type: 'success'
            });
          }else{
            this.$message.error('删除失败');
          }
        })
      },
      addUser(){
        this.form={id:0};
        this.dialogVisible=true;
      },
      updateUser(user){
        this.form=user;
        this.imageUrl=user.imgaddr;
        this.dialogVisible=true;
      },
      save(){
        let url = "http://localhost:10000/test/addUser";
        if(this.form.id > 0){
          url = "http://localhost:10000/test/updateUser";
        }
        this.$axios.post(url,this.form).then(respost => {
          if(respost.data.code == 200){
            this.getList();
            this.$message({
              message: '保存成功',
              type: 'success'
            });
          }else{
            this.$message.error('保存失败');
          }
          this.toclose();
        })
      },
      toclose(){
        this.dialogVisible=false;
        this.form={};
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;
      },
      getList() {
        this.$axios.post("http://localhost:10000/test/findUser", this.mydata).then(respost => {
          this.tableData = respost.data.list;
          this.total = respost.data.total;
          console.log(respost)
        })
      },
      handleSizeChange(val) {
        this.mydata.pageSize = val;
        this.getList();
      },
      handleCurrentChange(val) {
        this.mydata.page = val;
        this.getList();
      },
      onSubmit() {
        this.getList();
      },
      deleteUser(id){
        this.$axios.post("http://localhost:10000/test/deleteUser?id="+id).then(respost => {
          if(respost.data.code == 200){
            if((this.total-1) % this.mydata.pageSize == 0){
              this.mydata.page=this.mydata.page-1;
            }
            this.getList();
            this.$message({
              message: '删除成功',
              type: 'success'
            });
          }else{
            this.$message.error('删除失败');
          }
        })
      }
    }
  }
</script>

<style scoped>

</style>
