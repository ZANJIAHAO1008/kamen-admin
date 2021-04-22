<template>
  <div>
    <div class="container">
      <div class="handle-box">
        <el-button type="primary" @click="operation('add')">新增</el-button>
      </div>
      <el-table
          :data="radioData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
      >
        <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
        <el-table-column prop="title" label="标题"></el-table-column>
        <el-table-column label="视频地址" align="center" prop="radioUrl">
          <template  #default="scope">
<!--            <el-link v-if="scope.row.url" :href="scope.row.radioUrl" target="_blank"  type="danger">点这里查看...</el-link>-->
            <video  v-if="scope.row.url" :src="scope.row.radioUrl" width="200" height="200" controls="controls">
            </video>
            <span v-else>无视频，请先上传</span>
          </template>
        </el-table-column>
        <el-table-column prop="describes" label="描述"></el-table-column>
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
    <el-dialog :title="type" :visible.sync="editVisible" width="600px"  :before-close="clear">
      <el-form ref="form" :model="form" label-width="90px">
        <el-form-item label="标题：" prop="title">
          <el-input v-model="form.title" style="width: 220px;"></el-input>
        </el-form-item>
        <el-form-item label="封面上传" prop="imgUrl">
          <el-upload
              accept=".jpg,.png"
              action="http://146.56.235.246:8081/file/imageUpload"
              list-type="picture-card"
              :limit='1'
              :on-success="coverSuccess"
              name="image"
              ref="video"
              :clearFiles="clearFiles"
              :file-list="coverList"
              :on-remove="videoRemove">
            <i class="el-icon-plus"></i>
          </el-upload>
          <el-dialog :visible.sync="dialogVisible">
            <img width="100%" :src="dialogImageUrl" alt="">
          </el-dialog>
        </el-form-item>
        <el-form-item label="视频上传：" prop="video">
          <el-upload
              accept=".AVI,.mov,.rmvb,.rm,.FLV,.mp4,.3GP"
              ref="update"
              class="upload-demo"
              action="http://146.56.235.246:8081/file/videoUpload"
              :on-success="upSuccess"
              multiple
              name="video"
              :on-remove="handleRemove"
              :limit='1'
              :file-list="fileList">
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>
        <el-form-item label="描述：" prop="describes">
          <el-input
              type="textarea"
              rows="4"
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
  name: "manageRadio",
  data() {
    return {
      type:"",
      radioData:[],
      fileList:[],
      coverList:[],
      dialogImageUrl: '',
      dialogVisible: false,
      editVisible: false,
      form: {
        title:"",
        video:"",
        videoCover:"",
        describes:""

      },
      idx: -1,
    };
  },
  methods: {
    clearFiles () {
      this.$refs['video'].clearFiles();
    },

    upSuccess(val){
      this.form.video = val.data;
    },
    coverSuccess(val){
      this.form.videoCover = val.data;
    },
    getImage(){
      this.$axios({
        method: 'get',
        url: '/api/video/getVideoList',
        headers: {
          'Content-type': 'application/json;charset=UTF-8'
        },
        data:{}
      }).then(response => {
        if(response.data){
          this.radioData = response.data.data.map(v=>{
            if(v.url){
              v.radioUrl = 'http://146.56.235.246:8081/'+v.url;
            }
            if(v.videoCover){
              v.videoUrl = 'http://146.56.235.246:8081/'+v.videoCover;
            }
            return v;
          });
        }
      })
    },
    clear(){
      this.editVisible = false;
      this.fileList = [];
      this.coverList = [];
      this.clearFiles();
      this.$refs['form'].resetFields();
      this.form =  {
        title:"",
        video:"",
        videoCover: "",
        describes:""
      };
    },
    save(){
      if(this.type==='add'){
        this.addImage().then(()=>{
          this.$message({
            message: '新增成功',
            type: 'success'
          });
          this.clear();
          this.getImage();
        }).catch(err=>{
          this.clear();
          this.$message({
            message: err,
            type: 'warning'
          });
        })
      }else{
        this.saveEdit().then(()=>{
          this.$message({
            message: '修改成功',
            type: 'success'
          });
          this.clear();
          this.getImage();
        }).catch(err=>{
          this.clear();
          this.$message({
            message: err,
            type: 'warning'
          });
        })
      }
    },
    addImage(){//保存
      return new Promise(((resolve, reject) => {
        let dataWare = {
          title:this.form.title,
          url:this.form.video,
          describes:this.form.describes,
          videoCover:this.form.videoCover
        };
        this.$axios({
          method: 'post',
          url: '/api/video/addVideo',
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
              url: '/api/video/deleteVideo',
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
                this.getImage();
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
      console.log(row)
      this.form = {
        ...row
      };
      if(row.url){
        this.fileList = [
          { name:row.url,
            url:row.radioUrl
          }]
      }else{
        this.fileList = [];
      }

      if(row.videoCover){
        this.coverList = [
          {
            name:row.videoCover,
            url:row.videoUrl
          }
        ]
      }
      this.operation('edit');
    },
    handleRemove(file, fileList) {
      if(fileList.length==0){
        this.form.video = "";
      };
    },
    videoRemove(file, fileList) {
      if(fileList.length==0){
        this.form.videoCover = "";
      };
    },
    // 保存编辑
    saveEdit() {
      return new Promise(((resolve, reject) => {
        let dataWare = {
          id:this.form.id,
          title:this.form.title,
          url:this.form.video,
          describes:this.form.describes,
          videoCover:this.form.videoCover
        };
        this.$axios({
          method: 'put',
          url: '/api/video/editVideo',
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
  },
  watch:{
    name:{
      handler:'getImage',
      immediate:true
    }
  }
};
</script>

<style >
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
.el-upload--text {
  background-color: #fff;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  box-sizing: border-box;
  width: 360px;
   height:90px!important;
  text-align: center;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
</style>
