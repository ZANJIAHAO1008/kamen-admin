<template>
  <div>
    <div class="container">
      <div class="handle-box">
        <el-button type="primary" @click="operation('add')">新增</el-button>
      </div>
      <el-table
          :data="mapData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
      >
        <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
        <el-table-column prop="name" width="120" label="骑士名"></el-table-column>
        <el-table-column label="头像(查看大图)" align="center">
          <template #default="scope">
            <el-image
                class="table-td-thumb"
                :src="scope.row.image"
                :preview-src-list="[scope.row.image]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column show-overflow-tooltip width="100" prop="height" label="身高（单位cm）"></el-table-column>
        <el-table-column  show-overflow-tooltip width="100" prop="weight" label="体重（单位kg）"></el-table-column>
        <el-table-column show-overflow-tooltip width="100" prop="punchForce" label="拳力（单位t）"></el-table-column>
        <el-table-column show-overflow-tooltip width="100" prop="bouncingForce" label="跳跃力（单位m）"></el-table-column>
        <el-table-column show-overflow-tooltip width="100" prop="speed" label="速度（单位m/s）"></el-table-column>
        <el-table-column show-overflow-tooltip width="200" prop="knightPlay" label="骑士踢名"></el-table-column>
        <el-table-column show-overflow-tooltip width="300" prop="knightPlayDetails" label="骑士踢详情"></el-table-column>
        <el-table-column show-overflow-tooltip width="100" prop="changToThe" label="变身者"></el-table-column>
        <el-table-column  show-overflow-tooltip width="300" prop="altMode" label="变身形态"></el-table-column>
        <el-table-column show-overflow-tooltip width="300" prop="armed" label="武装"></el-table-column>
        <el-table-column fixed="right" label="操作" width="180" align="center">
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
    <el-dialog :title="type" :visible.sync="editVisible"  width="900px" top="1vh"  :before-close="clear" >
      <el-form ref="form" :inline="true" :model="form" label-width="120px">
        <div>
          <el-form-item label="骑士名：" prop="name">
            <el-input v-model="form.name" clearable style="width: 150px;"></el-input>
          </el-form-item>
          <el-form-item label="身高（cm）：" prop="height">
            <el-input v-model="form.height" clearable style="width: 150px;"></el-input>
          </el-form-item>
          <el-form-item label="体重（kg）：" prop="weight">
            <el-input v-model="form.weight" clearable style="width: 150px;"></el-input>
          </el-form-item>
        </div>
        <div>
          <el-form-item label="拳力（t）：" prop="punchForce">
            <el-input v-model="form.punchForce" clearable style="width: 150px;"></el-input>
          </el-form-item>
          <el-form-item label="跳跃力（m）：" prop="bouncingForce">
            <el-input v-model="form.bouncingForce" clearable style="width: 150px;"></el-input>
          </el-form-item>
          <el-form-item label="速度（m/s）：" prop="speed">
            <el-input v-model="form.speed" clearable style="width: 150px;"></el-input>
          </el-form-item>
        </div>
        <div>
          <el-form-item label="骑士踢名：" prop="knightPlay">
            <el-input v-model="form.knightPlay" clearable style="width: 150px;"></el-input>
          </el-form-item>
          <el-form-item label="骑士踢详情：" prop="knightPlayDetails">
            <el-input
                style="width: 431px;"
                type="textarea"
                :autosize="{ minRows: 2, maxRows: 4}"
                v-model="form.knightPlayDetails">
            </el-input>
          </el-form-item>
        </div>
        <div>
          <el-form-item label="变身者：" prop="changToThe">
            <el-input v-model="form.changToThe" clearable style="width: 150px;"></el-input>
          </el-form-item>
          <el-form-item label="变身形态：" prop="altMode">
            <el-input
                style="width: 431px;"
                type="textarea"
                :autosize="{ minRows: 2, maxRows: 4}"
                v-model="form.altMode">
            </el-input>
          </el-form-item>
        </div>
        <el-form-item label="武装：" prop="armed">
          <el-input
              style="width: 710px;"
              type="textarea"
              :autosize="{ minRows: 2, maxRows: 4}"
              v-model="form.armed">
          </el-input>
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
      name: "manageMap",
      data() {
        return {
          type:"",
          mapData:[],
          dialogImageUrl: '',
          dialogVisible: false,
          editVisible: false,
          fileList:[],
          form: {
            name:"",
            height:"",
            weight:"",
            punchForce:"",
            bouncingForce:"",
            speed:"",
            knightPlay:"",
            knightPlayDetails:"",
            changToThe:"",
            altMode:"",
            armed:"",
            imgUrl:"",
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
        getMap(){
          this.$axios({
            method: 'get',
            url: '/api/knight/getKnightList',
            headers: {
              'Content-type': 'application/json;charset=UTF-8'
            },
            data:{}
          }).then(response => {
            if(response.data){
              this.mapData = response.data.data.map(v=>{
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
            height:"",
            weight:"",
            punchForce:"",
            bouncingForce:"",
            speed:"",
            knightPlay:"",
            knightPlayDetails:"",
            changToThe:"",
            altMode:"",
            armed:"",
            imgUrl:"",
          };
        },
        save(){
          this.addImage().then(()=>{
            this.$message({
              message: this.type=='add'?'新增成功':"修改成功",
              type: 'success'
            });
            this.clear();
            this.getMap();
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
              name:this.form.name,
              height:this.form.height,
              weight:this.form.weight,
              punchForce:this.form.punchForce,
              bouncingForce:this.form.bouncingForce,
              speed:this.form.speed,
              knightPlay:this.form.knightPlay,
              knightPlayDetails:this.form.knightPlayDetails,
              changToThe:this.form.changToThe,
              altMode:this.form.altMode,
              armed:this.form.armed,
              imgUrl:this.form.imgUrl,
            };
            this.$axios({
              method: 'post',
              url: '/api/knight/saveKnight',
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
                  url: '/api/knight/deleteKnight',
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
                    this.getMap();
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
          handler:'getMap',
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
  ate>