<template>
  <div>
    <el-card>
      <CategorySelect @getCategoryId="getCategoryId" />
    </el-card>
    <el-card style="margin-top:20px">
      <!-- 展示平台售卖属性 -->
      <div v-show="isShowTable">
        <!-- 底部的结构 -->
        <el-button type="primary" icon="el-icon-plus" @click="changeShowTable" :disabled="!category3Id">添加属性</el-button>
        <!-- table表格 -->
        <el-table style="width: 100%;margin-top:10px" border :data="attrList">
          <el-table-column type="index" label="序号" width="80px" align="center"></el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="150px"></el-table-column>
          <el-table-column  label="属性值列表" width="width">
                <template slot-scope="{row,$index}">
                  <!-- :hit="true" :disable-transitions="true" (没看出来) -->
                  <el-tag type="danger" v-for="(attrValue,index) in row.attrValueList" :key="attrValue.id" style="margin:10px">{{attrValue.valueName}}</el-tag>
                </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="150px">
            <template slot-scope="{row,$index}">
              <el-button type="warning" icon="el-icon-edit" size="mini" @click="updateAttr(row)"></el-button>
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="delAttr(row)"></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 操作平台售卖属性 -->
      <div v-show="!isShowTable">
        <!-- 添加属性名 -->
        <el-form :inline="true" :model="attrInfo">
          <el-form-item label="属性名">
            <el-input placeholder="请你输入属性名" v-model="attrInfo.attrName" ></el-input>
          </el-form-item>
        </el-form>
        <!-- 添加属性值与取消按钮 -->
        <el-button type="primary" icon="el-icon-plus" @click="addAttr" :disabled="!attrInfo.attrName">添加属性值</el-button>
        <el-button @click="isShowTable = true">取消</el-button>
        <!-- 表格 -->
        <el-table
          style="width: 100%;margin:20px 0px" border :data="attrInfo.attrValueList" >
          <el-table-column type="index" label="序号" width="80px" align="center"></el-table-column>
          <el-table-column prop="prop" label="属性值名称" width="width">
            <template slot-scope="{ row, $index }">
              <el-input v-model="row.valueName" size="mini" v-show="row.flag" @blur="toLook(row)" @keyup.native.enter="toLook(row)" :ref="$index"></el-input>
              <span v-show="!row.flag" style="display:block" @click="toEdit(row, $index)" >
                {{ row.valueName }}
              </span>
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-button type="danger" size="mini" icon="el-icon-delete" @click="attrInfo.attrValueList.splice($index,1)"></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-button type="primary" @click="saveAttrInfo" :disabled="!attrInfo.attrValueList.length>0">保存</el-button>
        <el-button @click="isShowTable = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
//引入lodash/cloneDeep（按需引入）
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "Attr",
  //data函数当中不允许书写this.xxx
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      //属性列表的数据
      attrList:[],
      //控制组件显示
      isShowTable:true,
      attrInfo:{
        attrName:'',//属性名字
        attrValueList: [],
        categoryId: 0,
        categoryLevel: 3,
      },
      //数据结构
      // test:{
        // "attrName": "string",
        // "attrValueList": [
        //   {
        //     "attrId": 0,
        //     "id": 0,
        //     "valueName": "string"
        //   }
        // ],
        // "categoryId": 0,
        // "categoryLevel": 0,
        // "id": 0
      // },
    };
  },
  methods: {
    //父组件中的函数:(自定义事件的回调)
    getCategoryId({ categoryId, flag }) {
      if (flag == 1) {
        this.category1Id = categoryId;
        //清空父组件的数据
        this.category2Id = "";
        this.category3Id = "";
      } else if (flag == 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        //当用户选择完三级联动----父组件要发请求获取数据
        this.getAttrList();
      }
    },
    //获取属性列表的方法
    async getAttrList(){
      try {
        const {category1Id,category2Id,category3Id} = this;
        let result = await this.$API.attr.reqAttrList(category1Id,category2Id,category3Id);
        if(result.code==200){
          this.attrList = result.data;
       }
      } catch (error) {
        return error.message;
      }
    },
    //点击显示添加表格----
    changeShowTable(){
      //显示表格
      this.isShowTable = false;
      //再次点击添加一定记住清空数据，常识问题，呼吸般自然！！！！
      this.attrInfo = {
        attrName: "", //收集添加或者修改的属性值名字
        attrValueList: [], //收集相应的属性的属性值数组
        categoryId: this.category3Id, //三级分类id
        categoryLevel: 3
      };
    },
    //添加属性名按钮子（往数组里面推push一个空对象子，下面input是遍历出来的，多以直接出来一行空input待输入数据）
    addAttr(){
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id,
        valueName: "",
        //点击添加属性值按钮的时候，属性值对象身上添加了一个flag属性，用于控制每一个属性值当中是显示input|span
        //将来点击保存按钮的时候，向服务器发请求，flag干掉，因为服务器不需要这个参数
        //flag是不是响应式数据？答案：是！！！！！！！
        flag: true
      });
    },
    //查看状态：span
    toLook(row){
      //输入校验
      if(row.valueName.trim()==''){
        this.$message({
          message:'标签名不能为空，请输入合法标签名！',
          type: 'warning'
        });
        return;
      }
      let isRepeat = this.attrInfo.attrValueList.some(item =>
        row != item ? item.valueName === row.valueName : false
      );
      if (isRepeat) {
        this.$message({
          message:'标签名重复，请重新输入！',
          type: 'warning'
        });
        return;
      }
      row.flag = false;
    },
    //编辑状态：input
    toEdit(row,index){
      row.flag = true;
      //当flag变为true,input标签要上树【虚拟DOM->真实的DOM渲染，渲染很耗事件的】
      //切记：以后功的时候通过ref获取真实DOM，如果获取不到$nextTick
      //切记：以后功的时候，数据发生变化但是视图不更新$set
      this.$nextTick(() => {
        //确保DOM上树-渲染完毕了，而且DOM存在了
        this.$refs[index].focus();
      });
    },
    //编辑按钮子回调
    updateAttr(row){
      //显示添加|编辑的表单子
      this.isShowTable = false;
      //对象中有数组，所以避免数据在输入时就被同步修改，要使用深拷贝。浅拷贝不太行了！
      this.attrInfo = cloneDeep(row);
      //$set使用场景：将需要添加的某一属性（数据）变为响应式数据，而上面的响应式数组直接push就直接变成响应式数据啦。
      this.attrInfo.attrValueList.forEach((item)=>{
        //在给一个对象添加新的属性
        //以后工作的时候，你发现数据变了，但是视图不变【因为这条数据不是响应式的】$set|$nextTick
        //$nextTick理解为异步，等待DOM元素渲染完成后操作（确保可以操作到DOM）切记！！！！
        this.$set(item,'flag',false);
      })
    },
    //添加|修改的表单中的保存按钮子回调函数
    async saveAttrInfo(){
      //在传参数发请求之前，整理一波子数据
      //attrName不能为空，不需要提交flag字段，（用于input和span切换的标志)。
      //filter方法不会改变原数组，返回值为一个通过过滤条件的新数组，回调返回布尔值
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter((item)=>{
        if (item.attrName != '') {
          delete item.flag;
          return true;
        };
      })
      // console.log(this.attrInfo)
      // debugger;
      let res = await this.$API.attr.reqSaveAttrInfo(this.attrInfo);
      if (res.code == 200) {
        //成功返回展示数据页面，并重新获取数据
        this.isShowTable = true;
        //重新获取数据
        this.getAttrList();
      }
    },
    async delAttr({id}){
      // console.log(id);
      // debugger;
      let result = await this.$API.attr.reqDeleteAttr(id);
      if(result.code==200){
        this.getAttrList();
      }
    }
  }
};
</script>

<style scoped></style>
