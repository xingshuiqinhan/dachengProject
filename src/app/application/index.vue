<template>
<div id="applicationPage">
  <p class="tit">
    应用管理
  </p>
  <div class="handleBox clearfix">
    <div class="selectLeft">
          
    <el-select v-if="isSuper"  v-model="selectOPId" filterable placeholder="请选择" @change="changeOperator">
      <el-option v-for="item in OperatorList"  :label="item.OPName" :value="item.OPId">
    </el-option>
  </el-select>
  <el-button @click="addDialog = true;" type="primary"  class="addNewBtn">新增</el-button>
  
    </div>
 <div class="searchInfo">
      <el-date-picker
            v-model="starttime"
            type="datetime"
            :editable = "false"
            placeholder="选择开始时间">
      </el-date-picker>
       <el-date-picker
            v-model="endtime"
            type="datetime"
            :editable = "false"
            placeholder="选择截止时间">
      </el-date-picker>
      关键字：
      <el-input v-model="keywords" class="keywords" placeholder="请输入关键字"></el-input>
      <el-button type="primary" :disabled="disableSearch" @click="getAppList">搜索</el-button>
  </div>
  </div>
 
  <el-table :data="appData" style="width: 100%">
    <!-- <el-table-column prop="OPId" label="运营商ID"></el-table-column> -->
    <el-table-column prop="APName" label="应用名称"></el-table-column>
    <el-table-column prop="APId" label="应用ID"></el-table-column>
    
    <el-table-column prop="Type" label="应用协议类型"> </el-table-column>
    <el-table-column prop="Parameter" label="应用协议参数">
        <el-table-column label="ip地址" prop="Parameter.ip"></el-table-column>
        <el-table-column label="端口号" :center="true" prop="Parameter.port"></el-table-column>
        <el-table-column label="路由地址" prop="Parameter.path"></el-table-column>
    </el-table-column>
    <el-table-column prop="CreateTime" :formatter="formatDate"  label="创建时间"></el-table-column>
    <!-- <el-table-column prop="Validity"  label="有效期"></el-table-column> -->
    <el-table-column prop="Call"  label="调用总次数"></el-table-column>
    <el-table-column label="操作" width="150">
      <template slot-scope="scope" >       
        <el-button size="mini" @click="editCurApp(scope)">编辑</el-button>
        <el-button size="mini" type="danger" @click="confirmDelete(scope)" :disabled="disableDelete">删除</el-button>
      </template>
    </el-table-column>
  </el-table>


    <el-dialog title="新增应用信息" :visible.sync="addDialog" :before-close="closeDialog" :close-on-click-modal='false'>
      <div class="editDg">
        <el-table style="width: 100%" :data="[null]">
          <el-table-column label="应用名称">
                    <template slot-scope="scope">
                      <el-input v-model="curAppData.APName" placeholder="请输入内容"></el-input> 
                    </template> 
            </el-table-column>
            <el-table-column label="已有接口列表">
                    <template slot-scope="scope">
                     <el-select v-model="selectAppId" placeholder="请选择" @change="changeInterface">
                <el-option  label="请选择" value="" > </el-option>
                <el-option v-for="(item,index) in interfaces" :label="item.IName" :value="index">
                </el-option>
              </el-select>
                    </template> 
            </el-table-column>
            <el-table-column  label="应用协议类型">
              <template slot-scope="scope">
              <el-select v-model="curAppData.Type" placeholder="请选择">
                <el-option v-for="item in applyType" :label="item" :value="item" >
                </el-option>
              </el-select>
              </template>
            </el-table-column>
            <el-table-column  label="应用协议参数">
                <el-table-column label="ip地址">
                    <template slot-scope="scope">
                      <!-- <el-input v-model="curAppData.Parameter.ip" placeholder="请输入内容"></el-input>  -->

  <el-select v-model="curAppData.Parameter.ip" placeholder="请选择" allow-create 
    filterable>
      <el-option v-for="item in ids" :value="item">
    </el-option>
  </el-select>
                    </template> 
                </el-table-column>
                <el-table-column label="端口号">
                    <template slot-scope="scope">
                      <el-input v-model="curAppData.Parameter.port" placeholder="请输入内容"></el-input>  
                    </template> 
                </el-table-column>
                <el-table-column label="路由地址" v-if="curAppData.Type=='http' ">
                    <template slot-scope="scope" >
                     


          <el-select v-model="curAppData.Parameter.path" placeholder="请选择" allow-create 
              filterable>
                <el-option v-for="item in paths" :key="item" :value="item">
              </el-option>
            </el-select>
                    </template>               
                </el-table-column>
            </el-table-column>
            <el-table-column  label="操作" >
              <template slot-scope="scope">
                <el-button type="primary" @click="addNewApp" :disabled="disableAdd">新增</el-button>
              </template>
            </el-table-column>
        </el-table>
      </div>
    </el-dialog>
  <el-dialog title="编辑应用信息" :before-close="closeDialog" :visible.sync="editDialog" :close-on-click-modal='false'>
      <div class="editDg">
        <el-table style="width: 100%" :data="[null]">
            <el-table-column label="应用名称">
                    <template slot-scope="scope">
                      <el-input v-model="curAppData.APName" placeholder="请输入内容"></el-input> 
                    </template> 
            </el-table-column>


 <el-table-column label="已有接口列表">
                    <template slot-scope="scope">
                     <el-select v-model="selectAppId" placeholder="请选择" @change="changeInterface">
                <el-option  label="请选择" value="" > </el-option>
                <el-option v-for="(item,index) in interfaces" :label="item.IName" :value="index">
                </el-option>
              </el-select>
                    </template> 
            </el-table-column>
            <el-table-column  label="应用协议类型" >
              <template slot-scope="scope">
              <el-select v-model="curAppData.Type" placeholder="请选择">
                <el-option
                  v-for="item in applyType"
                  
                  :label="item"
                  :value="item">
                </el-option>
              </el-select>
              </template>
            </el-table-column>
            <el-table-column  label="应用协议参数" >
                <el-table-column label="ip地址">
                    <template slot-scope="scope">
                       <el-select v-model="curAppData.Parameter.ip" placeholder="请选择" allow-create  filterable>
                                <el-option v-for="item in ids"  :value="item">
                              </el-option>
                            </el-select>
                    </template> 
                </el-table-column>
                <el-table-column label="端口号">
                    <template slot-scope="scope">
                      <el-input v-model="curAppData.Parameter.port" placeholder="请输入内容"></el-input>  
                    </template> 
                </el-table-column>
                <el-table-column label="路由路径" v-if="curAppData.Type=='http' ">
                    <template slot-scope="scope" >
                       <el-select v-model="curAppData.Parameter.path" placeholder="请选择" allow-create 
              filterable>
                <el-option v-for="item in paths"  :value="item">
              </el-option>
            </el-select>
                    </template>               
                </el-table-column>
            </el-table-column>
            <el-table-column  label="操作" >
              <template slot-scope="scope">
                <el-button type="primary" @click="updateCurApp" :disabled="disableUpdate">保存</el-button>
              </template>
            </el-table-column>
        </el-table>
      </div>
    </el-dialog>
<el-pagination background layout="prev, pager, next" :total="listCount"
@current-change="changePage" :current-page="currentPage" :page-size="pageSize"
></el-pagination>
</div>
</template>
<style>
#applicationPage .addNewBtn {
  top: 55px;
}
#applicationPage .el-dialog {
  width: 80%;
}
#applicationPage .handleBox {
  height: 50px;
}
</style>
<script>
import {
  showErrMsg,
  formatDate,
  deepClone,
  isPattern,
  pagesNum
} from "../common/utils.js";
export default {
  data() {
    return {
      interfaces: [],
      selectAppId: "",
      appIds: [],
      disableSearch: false,
      keywords: "",
      starttime: "",
      endtime: "",
      ids: [111, 222, 333],
      paths: ["path1", "path2"],
      isSuper: true, //是否超级管理员
      selectOPId: "", //选中的运营商OPId
      OperatorList: [], //运营商列表
      disableAdd: false,
      disableUpdate: false,
      disableDelete: false,
      currentPage: 1,
      pageSize: pagesNum,
      listCount: null,
      addDialog: false,
      editDialog: false,
      applyType: ["http", "tcp", "udp"],
      curAppData: {
        OPId: "",
        APId: "",
        APName: "",
        Type: "",
        Parameter: {
          ip: "",
          port: "",
          path: ""
        },
        CreateTime: "",
        Validity: "",
        Call: ""
      },
      currentNum: 1,
      appData: []
    };
  },
  components: {
    SearchHeader: require("../components/searchHeader.vue")
  },
  created() {
    this.isSuper = this.$store.state.is_super;
    this.selectOPId = this.$store.state.OPId;
    this.getHistoryIpPath();
    if (this.isSuper) {
      this.getOperatorList();
    }
    this.getAppList();
    this.getInterfaces();
  },

  methods: {
    // chooseIpPort() {
    //   console.log(this.selectAppId);
    // },
    // searchHeaderList() {
    //   console.log("chaxun");
    // },
    confirmDelete(scope) {
      let that = this;
      this.$alert("", "是否删除", {
        confirmButtonText: "确定",
        callback: action => {
          if (action == "cancel") return;
          that.deleteCurApp(scope);
        }
      });

      //deleteCurApp(scope);
    },
    //格式化时间
    formatDate(t) {
      return formatDate(parseInt(t.CreateTime));
    },
    changeInterface() {
      this.curAppData.Parameter.ip = this.interfaces[this.selectAppId].IIP;
      this.curAppData.Parameter.port = this.interfaces[this.selectAppId].IPort;

      this.curAppData.Parameter.path = this.interfaces[this.selectAppId].IPath;
    },
    getInterfaces() {
      let that = this;
      $.ajax({
        type: "post",
        data: {
          token: this.$store.state.usr_token
        },
        url: "/application/getclientjson",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          that.interfaces = d.data || [];
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          showErrMsg(that, textStatus);
        }
      });
    },
    getHistoryIpPath() {
      let that = this;
      $.ajax({
        type: "post",
        data: {
          token: this.$store.state.usr_token
        },
        url: "/application/gethistory",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          that.ids = d.data.Ip || [];
          that.paths = d.data.Path || [];
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          showErrMsg(that, textStatus);
        }
      });
    },
    //跳转到第几页
    changePage(currentPage) {
      this.currentPage = currentPage;
      this.getAppList();
    },
    changeOperator() {
      this.currentPage = 1;
      this.getAppList();
    },
    //编辑当前app信息
    editCurApp(scope) {
      this.currentNum = scope.$index;
      this.editDialog = true;
      this.curAppData = deepClone(this.appData[this.currentNum]);
    },
    //关闭弹框
    closeDialog() {
      this.selectAppId = "";
      this.editDialog = false;
      this.addDialog = false;
      this.curAppData = {
        OPId: "Admin",
        APId: "",
        APName: "",
        Type: "",
        Parameter: {
          ip: "",
          port: "",
          path: ""
        },
        CreateTime: "",
        Validity: "",
        Call: ""
      };
    },
    //添加新应用
    addNewApp() {
      if (!isPattern(this.curAppData.APName, this, "应用名称")) return;
      var that = this;
      that.disableAdd = true;
      this.curAppData.OPId = this.selectOPId;

      $.ajax({
        type: "post",
        data: {
          Application: this.curAppData,
          token: this.$store.state.usr_token
        },
        url: "/application/add",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          that.disableAdd = false;
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          that.getAppList();
          that.closeDialog();
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          that.disableAdd = false;
          showErrMsg(that, textStatus);
        }
      });
    },

    //删除选中应用
    deleteCurApp(scope) {
      var that = this;
      that.disableDelete = true;
      $.ajax({
        type: "post",
        data: {
          APId: this.appData[scope.$index].APId,
          token: this.$store.state.usr_token
        },
        url: "application/remove",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          that.disableDelete = false;
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          that.$message({
            showClose: true,
            message: "删除成功",
            duration: 1500
          });
          that.getAppList();
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          that.disableDelete = false;
          showErrMsg(that, textStatus, "删除失败");
        }
      });
    },
    //修改选中应用
    updateCurApp() {
      if (!isPattern(this.curAppData.APName, this, "应用名称")) return;
      var that = this;
      that.disableUpdate = true;
      $.ajax({
        type: "post",
        data: {
          Application: this.curAppData,
          token: this.$store.state.usr_token
        },
        url: "application/update",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          that.disableUpdate = false;
          that.getAppList();
          that.closeDialog();
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          that.disableUpdate = false;
          showErrMsg(that, textStatus);
        }
      });
    },

    //获取应用列表
    getAppList() {
      var that = this;
      this.disableSearch = true;
      let starttime = new Date(this.starttime).getTime();
      let endtime = new Date(this.endtime).getTime();
      $.ajax({
        type: "post",
        data: {
          starttime: starttime,
          endtime: endtime,
          key: this.keywords,
          OPId: this.selectOPId,
          Index: this.currentPage,
          PageSize: parseInt(this.pageSize),
          token: this.$store.state.usr_token
        },
        url: "/application/getlist_index",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          that.disableSearch = false;
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          if (d.code == 99) {
            that.appData = [];
            return;
          }

          that.appData = d.data["List"] || [];
          that.listCount = d.data["Count"];
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          that.disableSearch = false;
          showErrMsg(that, textStatus, "请求失败" + XMLHttpRequest.status);
        }
      });
    },
    //获取运营商列表
    getOperatorList() {
      var that = this;
      $.ajax({
        type: "post",
        data: {
          opid: this.selectOPId,
          token: this.$store.state.usr_token
        },
        url: "/operators/getlist",
        dataType: "json",
        timeout: 20000,
        success: function(d) {
          if (d.code == 55) {
            showErrMsg(that, 55, "token验证失效，请重新登录");
            that.$router.push({ path: "/login" });
            return;
          }
          if (d.code == 99) {
            that.OperatorList = [];
            return;
          }
          that.OperatorList = d.data || [];
        },
        error: function(XMLHttpRequest, textStatus, errorThrown) {
          showErrMsg(that, textStatus, "请求失败" + XMLHttpRequest.status);
        }
      });
    }
  }
};
</script>