<template>
  <div>
    <div class="container">
      <div class="handle-box">
        <el-button type="primary" @click="operation('add')">新增</el-button>
      </div>
      <el-table
          :data="chronicleData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
      >
        <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
        <el-table-column prop="name" label="片名"></el-table-column>
        <el-table-column label="播出时间" prop="airTime"></el-table-column>
        <el-table-column label="头像(查看大图)" align="center">
          <template #default="scope">
            <el-image
                class="table-td-thumb"
                :src="scope.row.image"
                :preview-src-list="[scope.row.image]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column show-overflow-tooltip width="600" prop="describes" label="描述"></el-table-column>
        <el-table-column label="操作" width="180" align="center">
          <template #default="scope">
            <el-button
                type="text"
                icon="el-icon-edit"
                @click="handleEdit(scope.row)"
            >编辑</el-button>
            <el-button
                type="text"
                icon="el-icon-delete"
                class="red"
                @click="handleDelete( scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 编辑弹出框 -->
    <el-dialog :title="type"  :visible.sync="editVisible" width="600px"  :before-close="clear">
      <el-form ref="form" :model="form" label-width="84px">
        <el-form-item label="片名：" prop="name">
          <el-input v-model="form.name" style="width: 220px;"></el-input>
        </el-form-item>
        <el-form-item label="播出时间：" prop="airTime">
          <el-date-picker
              v-model="form.airTime"
              type="date"
              placeholder="选择日期"
              value-format="yyyy-MM-dd">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="图片上传" prop="imgUrl">
          <el-upload
              accept=".jpg,.png"
              action="http://146.56.235.246:8081/file/imageUpload"
              list-type="picture-card"
              :limit='1'
              :on-success="upSuccess"
              name="image"
              ref="update"
              :clearFiles="clearFiles"
              :file-list="fileList"
              :on-remove="handleRemove">
            <i class="el-icon-plus"></i>
          </el-upload>
          <el-dialog :visible.sync="dialogVisible">
            <img width="100%" :src="dialogImageUrl" alt="">
          </el-dialog>
        </el-form-item>
        <el-form-item label="描述：" prop="describes">
          <el-input
              :rows="4"
              type="textarea"
              placeholder="请输入内容"
              v-model="form.describes"
              show-word-limit
          ></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
                <span class="dialog-footer">
                    <el-button @click="clear">取 消</el-button>
                    <el-button type="primary" @click="save">确 定</el-button>
                </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
// import { fetchData } from "../api/index";
export default {
  name: "manageChronicle",
  data() {
    return {
      type:"",
      chronicleData:[],
      dialogImageUrl: '',
      dialogVisible: false,
      editVisible: false,
      fileList:[],
      form: {
        name:"",
        airTime:"",
        imgUrl:"",
        describes:""
      },
      idx: -1,
    };
  },
  methods: {
    handleRemove(file, fileList) {
      if(fileList.length==0){
        this.form.imgUrl = "";
      }
    },
    clearFiles () {
      this.$refs['update'].clearFiles();
    },
    getChronicle(){
      this.$axios({
        method: 'get',
        url: '/api/chronicle/getChronicleList',
        headers: {
          'Content-type': 'application/json;charset=UTF-8'
        },
        data:{}
      }).then(response => {
        if(response.data){
          this.chronicleData = response.data.data.map(v=>{
            if(v.imgUrl){
              v.image = 'http://146.56.235.246:8081/'+v.imgUrl;
            }
            return v;
          });
        }
      })
    },
    clear(){
      this.editVisible = false;
      this.fileList = [];
      this.clearFiles();
      this.$refs['form'].resetFields();
      this.form =  {
        name:"",
            airTime:"",
            imgUrl:"",
            describes:""
      };
    },
    save(){
      this.addImage().then(()=>{
        this.$message({
          message: this.type=='add'?'新增成功':"修改成功",
          type: 'success'
        });
        this.clear();
        this.getChronicle();
      }).catch(err=>{
        this.clear();
        this.$message({
          message: err,
          type: 'warning'
        });
      })
    },
    addImage(){//保存
      return new Promise(((resolve, reject) => {
        let dataWare = {
          id:this.form.id,
          airTime:this.$moment(this.form.airTime).format('YYYY-MM-DD'),
          name:this.form.name,
          imgUrl:this.form.imgUrl,
          describes:this.form.describes,
        };
        this.$axios({
          method: 'post',
          url: '/api/chronicle/addChronicle',
          headers: {
            'Content-type': 'application/json;charset=UTF-8'
          },
          data:JSON.stringify(dataWare)
        }).then(response => {
          if(response.data.code===200){
            resolve(response.data.data)
          }
        }).catch(function (error) {
          reject(error)
        });
      }))
    },
    operation(type){
      type=='add'?this.type = 'add':this.type = 'edit';
      this.editVisible = true;
    },
    // 获取 easy-mock 的模拟数据
    getData() {

    },
    // 删除操作
    handleDelete(row) {
      // 二次确认删除
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
          .then(() => {
            this.$axios({
              method: 'delete',
              url: '/api/chronicle/deleteChronicle',
              headers: {
                'Content-type': 'application/json;charset=UTF-8'
              },
              params:{
                id:row.id
              }
            }).then(response => {
              if(response.data.code===200){
                this.$message({
                  message: '删除成功',
                  type: 'success'
                });
                this.getChronicle();
              }
            }).catch(function (error) {
              this.$message({
                message: error,
                type: 'warning'
              });
            });

          })
          .catch(() => {});
    },
    // 编辑操作
    handleEdit(row) {
      this.form = {
        ...row
      };
      if(row.imgUrl){
        this.fileList = [
          { name:row.imgUrl,
            url:row.image
          }]
      }else{
        this.fileList = [];
      }
      this.operation('edit');
    },
    upSuccess(val){
      this.form.imgUrl = val.data;
    },
    // // 保存编辑
    // saveEdit() {
    //   return new Promise(((resolve, reject) => {
    //     let dataWare = {
    //       id:this.form.id,
    //       title:this.form.title,
    //       imgUrl:this.form.imgUrl,
    //       describes:this.form.describes,
    //     };
    //     this.$axios({
    //       method: 'post',
    //       url: '/api/images/addImage',
    //       headers: {
    //         'Content-type': 'application/json;charset=UTF-8'
    //       },
    //       data:JSON.stringify(dataWare)
    //     }).then(response => {
    //       if(response.data.code===200){
    //         resolve(response.data.data)
    //       }
    //     }).catch(function (error) {
    //       reject(error)
    //     });
    //   }))
    // },
  },
  watch:{
    name:{
      handler:'getChronicle',
      immediate:true
    }
  }
};
</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}
.table {
  width: 100%;
  font-size: 14px;
}
.red {
  color: #ff0000;
}
.mr10 {
  margin-right: 10px;
}
.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>
