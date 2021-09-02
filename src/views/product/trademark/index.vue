<template>
  <div>
    <el-button type="primary" icon="el-icon-plus" @click="addTradeMark">添加</el-button>
    <el-table style="width:100%;margin:30px 0px" border :data="records">
      <el-table-column
        label="序号"
        width="80px"
        type="index"
        align="center"
      ></el-table-column>
      <el-table-column prop="tmName" label="品牌名称" width="width"></el-table-column>
      <el-table-column prop="tmName" label="品牌Logo" width="width">
        <template slot-scope="{row,incdex}">
          <img :src="row.logoUrl" style="width:100px;height=100px">
        </template>
      </el-table-column>
      <el-table-column prop="tmName" label="操作" width="width">
        <template>
          <el-button type="warning" size="mini" icon="el-icon-edit" >修改</el-button>
          <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
        </template>
      </el-table-column>
      </el-table>
    <el-pagination 
      style="textAlign:center"
      :total="total" 
      :current-page="page" 
      :page-size="limit" 
      :page-sizes="[3, 5, 10]" 
      layout="prev, pager, next, jumper,->,sizes,total"
      @size-change="handleSizeChange"
      @current-change="getData"
    ></el-pagination>
    <!-- 点击添加弹出对话框子 -->

    <el-dialog title="添加品牌" :visible.sync="dialogFormVisible">
      <el-form style="width:80%" :model="tmForm">
        <el-form-item label="品牌名称" label-width="80px">
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌logo" label-width="80px">
          <!--  
          -->
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
            :show-file-list="false">
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name:'Trademark',
  data(){
    return {
      //初始化数据
      page: 1, //代表当前第几页
      limit: 3, //代表每一个展示多少条数据
      total: 0, //分页器一共要展示的数据,
      records: [], //表格展示产品的数据
      dialogFormVisible: false, //控制对话框显示与隐藏
      //收集添加品牌的数据
      //imageUrl:'',
      tmForm:{
        tmName:'',
        logoUrl:''
      },
    }
  },
  mounted(){
    //获取数据
    this.getData()
  },
  methods:{
    //获取品牌列表数据
    async getData(pages = 1) {
      //重新修改page
      this.page = pages;
      //解构
      const { page, limit } = this;
      let res = await this.$API.trademark.reqTradeMarkList(page, limit);
      if (res.code == 200) {
        this.total = res.data.total;
        this.records = res.data.records;
      }
    },
    //分页器
    //当每一页展示的数据条数发生变化的时候触发
    handleSizeChange(limit) {
      this.limit = limit;
      this.getData();
    },
    addTradeMark(){
      //对话跨显示出来
      this.dialogFormVisible = true;
    },
        //上传图片相关的
    handleAvatarSuccess(res, file) {
      //收集图片信息，添加品牌的时候收集的数据
      this.tmForm.logoUrl = res.data;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    async addOrUpdateTradeMark(){
      console.log(1)
      this.dialogFormVisible = false;
      //发请求---添加品牌请求
      console.log(this.tmForm)
      let result  = await this.$API.trademark.reqAddOrUpdateTradeMark(this.tmForm);
      //如果添加成功，需要再次发请求，获取产品列表数据
      if(result.code==200){
         this.$message({
          showClose: true,
          message:this.tmForm.id?'修改成功':'新增成功',
          type: 'success'
        });
         this.getData();
      }
    }
  }
}
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
