<template>
  <div class="content-view">
    <!--表格-->
    <el-table border :data="tempList" style="width: 100%" @expand-change="handleMange">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="table-expand">
            <el-form-item label="项目ID:">
              <span>{{ props.row.projectId }}</span>
            </el-form-item>
            <el-form-item label="项目名称:">
              <span>{{ props.row.projectName }}</span>
            </el-form-item>
            <el-form-item label="负责人:">
              <span>{{ userInfo.userName }}</span>
            </el-form-item>
            <el-form-item label="学号:">
              <span>{{ props.row.userId }}</span>
            </el-form-item>
            <el-form-item label="联系方式:">
              <span>{{ userInfo.userPhone }}</span>
            </el-form-item>
            <el-form-item label="项目等级:">
              <span v-if="props.row.grade === 1">校级</span>
              <span v-if="props.row.grade === 2">省级</span>
              <span v-if="props.row.grade === 3">国家级</span>
              <span v-if="props.row.grade === 0">无</span>
            </el-form-item>
            <el-form-item label="指导老师1:">
              <span v-if="props.row.oneId !== null">{{props.row.oneId}}</span>
              <span v-else>无</span>
            </el-form-item>
            <el-form-item label="指导老师2:">
              <span v-if="props.row.twoId !== null">{{props.row.twoId}}</span>
              <span v-else>无</span>
            </el-form-item>
            <el-form-item label="立案申请:">
              <span v-if="props.row.sreport == 1" style="color:green">已提交</span>
              <span v-else style="color:red">未提交</span>
            </el-form-item>
            <el-form-item label="中期报告:">
              <span v-if="props.row.mreport == 1" style="color:green">已提交</span>
              <span v-else style="color:red">未提交</span>
            </el-form-item>
            <el-form-item label="结案报告:">
              <span v-if="props.row.freport == 1" style="color:green">已提交</span>
              <span v-else style="color:red">未提交</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <template>
        <el-table-column label="项目名称" width="200" prop="projectName"></el-table-column>
        <el-table-column label="项目负责人学号" width="130" prop="userId"></el-table-column>
        <el-table-column label="所属学院" width="170" prop="collegeId" :formatter="GetCollegeName"></el-table-column>
        <el-table-column label="项目等级" width="80" align="center">
          <template slot-scope="scope">
            <span v-if="scope.row.grade === 1">校级</span>
            <span v-if="scope.row.grade === 2">省级</span>
            <span v-if="scope.row.grade === 3">国家级</span>
            <span v-if="scope.row.grade === 0">无</span>
          </template>
        </el-table-column>
        <el-table-column label="中期报告" width="80" align="center">
          <template slot-scope="scope">
            <span v-if="scope.row.mreport == 1" style="color:green">已提交</span>
            <span v-else style="color:red">未提交</span>
          </template>
        </el-table-column>
      </template>
      <el-table-column label="中期报告操作" align="center">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="mReport(scope.$index, scope.row)">查看</el-button>
          <el-button
            type="primary"
            size="small"
            @click="mReportApproval(scope.$index, scope.row)"
          >审核</el-button>
          <el-button type="primary" size="small" @click="setExpert(scope.row)">指派专家</el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="currentPage"
        :page-sizes="[1, 5, 10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>

    <div class="sumApp">
      <el-button type="primary" size="mini" @click="sumReport">汇总报表</el-button>
      <el-button type="primary" size="mini" @click="appoint">批量指派</el-button>
    </div>
    <el-dialog
      v-model="appointedForm"
      :visible.sync="appointedForm"
      class="midReport"
      title="中期报告专家指派"
    >
      <el-form style="margin:0">
        <el-form-item label="可指派列表:" label-width="120px" style="width:102%">
          <el-table
            :data="appointList"
            ref="appointList"
            height="250"
            border
            style="width: 100%;font-size:12px"
            @selection-change="handleSelectionChange"
          >
            <el-table-column type="selection"></el-table-column>

            <el-table-column prop="projectName" label="文件名" width="280"></el-table-column>
            <el-table-column label="负责人" width="80" prop="userName"></el-table-column>
            <el-table-column label="指导老师" width="80" prop="teacherName"></el-table-column>
            <el-table-column label="中期报告" width="100">
              <template slot-scope="scope">
                <span v-if="scope.row.submit === 0" style="color:red">未提交</span>
                <span v-if="scope.row.submit === 1" style="green">已提交</span>
              </template>
            </el-table-column>
          </el-table>
        </el-form-item>

        <el-form-item label="指派专家:" label-width="100px" class="appoint">
          <el-dropdown @command="handleCommand" trigger="click" placement="bottom-start">
            <span class="el-dropdown-link" v-if="chooseExpert == ''">
              专家列表
              <i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <span class="el-dropdown-link" v-else>
              {{chooseExpert}}
              <i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                :command="(item.userId)"
                v-for="item in expertList"
                :key="item.index"
              >ID:{{item.userId}},姓名:{{ item.userName }},联系方式：{{item.phone}}</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitAppoint">确认指派</el-button>
      </div>
    </el-dialog>

    <el-dialog
      v-model="dialogFormVisibleNew"
      :visible.sync="dialogFormVisibleNew"
      class="midReport"
      title="审核中期报告"
    >
      <el-form class="mReportForm">
        <el-form-item label="项目名称:" label-width="100px">
          <span>{{midReportNew.projectName}}</span>
        </el-form-item>
        <el-form-item label="项目编号:" label-width="100px">
          <span>{{midReportNew.projectId}}</span>
        </el-form-item>
        <el-form-item label="学生学号" label-width="100px">
          <span>{{midReportNew.userId}}</span>
        </el-form-item>
        <el-form-item label="评语" label-width="100px">
          <el-input v-model="comment"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="pass" type="primary">通过</el-button>
        <el-button @click="Back" type="primary">退回</el-button>
      </div>
    </el-dialog>
    <el-dialog
      v-model="dialogFormVisible"
      :visible.sync="dialogFormVisible"
      class="midReport"
      title="中期报告详情"
    >
      <template>
        <el-form class="mReportForm" :model="midReport">
          <el-form-item label="项目名称:" label-width="100px">
            <span>{{projectName}}</span>
          </el-form-item>
          <el-form-item label="中期报告简介:" label-width="100px">
            <span v-if="changeContentValue == false">
              <span>{{content}}</span>
              <a href="javascript:;" @click="changeContentValue = true" class="changeContent">修改</a>
            </span>
            <el-form class="contentForm" v-else>
              <el-input v-model="content"></el-input>
              <el-button type="primary" size="mini" @click="changeContent">确定</el-button>
            </el-form>
          </el-form-item>
          <el-form-item label="导师评议:" label-width="100px" class="midDisscuss">
            <span v-if="midReport.tapproval === 0">未审核</span>
            <span v-if="midReport.tapproval === 1" style="color:red">不通过</span>
            <span v-if="midReport.tapproval === 2" style="color:green">已通过</span>
            <span v-if="midReport.tapproval === 3" style="color:orange">退回修改</span>
          </el-form-item>
          <el-form-item label="学院评议:" label-width="100px" class="midDisscuss">
            <span v-if="midReport.capproval === 0">未审核</span>
            <span v-if="midReport.capproval === 1" style="color:red">不通过</span>
            <span v-if="midReport.capproval === 2" style="color:green">已通过</span>
            <span v-if="midReport.capproval === 3" style="color:blue">退回学生</span>
            <span v-if="midReport.capproval === 4" style="color:orange">退回导师</span>
          </el-form-item>
          <el-form-item label="大创管理评议:" label-width="100px" class="midDisscuss">
            <span v-if="midReport.sapproval === 0">未审核</span>
            <span v-if="midReport.sapproval === 1" style="color:red">不通过</span>
            <span v-if="midReport.sapproval === 2" style="color:green">已通过</span>
            <span v-if="midReport.sapproval === 3" style="color:orange">退回修改</span>
          </el-form-item>
          <el-form-item label="评审专家评议:" label-width="100px" class="midDisscuss">
            <span v-if="midReport.eapproval === 0">未审核</span>
            <span v-if="midReport.eapproval === 1" style="color:red">不通过</span>
            <span v-if="midReport.eapproval === 2" style="color:green">已通过</span>
            <span v-if="midReport.eapproval === 3" style="color:orange">暂缓通过</span>
          </el-form-item>
          <el-form-item label="导师评语:" label-width="100px">
            <span v-if="midReport.tcomment">{{midReport.tcomment}}</span>
            <span v-else>无</span>
          </el-form-item>
          <el-form-item label="学院评语:" label-width="100px">
            <span v-if="midReport.ccomment">{{midReport.ccomment}}</span>
            <span v-else>无</span>
          </el-form-item>
          <el-form-item label="大创管理评语:" label-width="100px">
            <span v-if="midReport.scomment">{{midReport.scomment}}</span>
            <span v-else>无</span>
          </el-form-item>
          <el-form-item label="评审专家:" label-width="100px">
            <span v-if="midReport.expertName">{{midReport.expertName}}</span>
            <span v-else>无</span>
          </el-form-item>
          <el-form-item label="评审专家评语:" label-width="100px">
            <span v-if="midReport.ecomment">{{midReport.ecomment}}</span>
            <span v-else>无</span>
          </el-form-item>
          <!-- <el-form-item
            label="上传文件:"
            label-width="100px"
            enctype="multipart/form-data"
            style="margin:0"
          >
            <el-upload
              class="upload-demo"
              ref="upload"
              action
              multiple
              :limit="1"
              :http-request="beforeUpload"
              :show-file-list="false"
            >
              <el-button slot="trigger" size="mini" type="primary">选取文件</el-button>
              <div slot="tip" class="el-upload__tip" style="margin:0">文件不超过5MB</div>
              <el-button
                type="primary"
                size="mini"
                style="margin-left:10px"
                @click="submitUpload()"
              >上传</el-button>
              <p
                v-if="fileShow === true"
                style="font-size:12px;color:#ccc;margin:0"
              >{{ mfileNewName }}</p>
            </el-upload>
          </el-form-item>-->
          <el-form-item label="文件列表:" label-width="100px" style="width:90%">
            <el-table :data="mfiles" height="250" border style="width: 100%;font-size:12px">
              <el-table-column prop="fname" label="文件名" width="210"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-link
                    tag="a"
                    target="_blank"
                    :href="'http://47.113.80.250:8012/onlinePreview?url='+scope.row.furl"
                  >
                    <el-button size="mini" type="primary">预览</el-button>
                  </el-link>
                  <el-button
                    type="primary"
                    size="mini"
                    style="margin-left:10px"
                    @click="download(scope.row)"
                  >下载</el-button>
                  <el-popconfirm
                    confirmButtonText="确认"
                    cancelButtonText="不了"
                    icon="el-icon-info"
                    title="确定删除该文件吗？"
                    style="margin-left:10px;"
                    @onConfirm="deleteFile(scope.$index,scope.row)"
                  >
                    <el-button slot="reference" size="mini" type="danger">删除</el-button>
                  </el-popconfirm>
                </template>
              </el-table-column>
            </el-table>
          </el-form-item>
        </el-form>
      </template>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogFormVisible = false">确定</el-button>
      </div>
    </el-dialog>
    <expertTable
      :isShow="isShow"
      class="eTable"
      :expertData="expert"
      @cancel="cancel"
      :setExpertPid="setExpertPid"
    ></expertTable>
    <div class="login-bg" v-show="isShow"></div>
  </div>
</template>

<script>
import { request } from "../../network/request/request";
import { mReportApproval } from "../../network/request/mReportApproval";
import {
  getmReport,
  findmReport,
  getAllExpert
} from "../../network/request/getmReport";
import expertTable from "./expertTable";

//  var fileDownload = require('js-file-download')
export default {
  data() {
    return {
      //项目数据
      tableData: [],
      userId: "",
      dialogFormVisible: false,
      dialogFormVisibleNew: false,
      baseUrl: "",
      userInfo: [],
      total: 0,
      tempList: [],
      currentPage: 1,
      pageSize: 10,
      midReport: [],
      midReportNew: [],
      projectName: "",
      mfiles: [],
      content: "",
      changeContentValue: false,
      fileShow: false,
      mfileNewName: "",
      mfileForm: new FormData(),
      comment: "",
      row: {},
      collegeList: [
        { id: 1, name: "计算机科学与工程学院" },
        { id: 2, name: "政法学院" },
        { id: 3, name: "电子信息与电气工程学院" },
        { id: 4, name: "地理与旅游学院" },
        { id: 5, name: "数学与大数据学院" },
        { id: 6, name: "化学与材料工程学院" },
        { id: 7, name: "建筑与土木工程学院" },
        { id: 8, name: "旭日广东服装学院" },
        { id: 9, name: "生命科学学院" },
        { id: 10, name: "经济管理学院" },
        { id: 11, name: "体育学院" }
      ],
      isShow: false,
      expert: [],
      setExpertPid: [],
      getAllList: [],
      appointList: [],
      appointedForm: false,
      expertList: [],
      chooseExpert: "",
      chooseReport: [],
      expertId: "",
      reportId: 0,
      setReportId: 0
    };
  },
  components: {
    expertTable
  },
  created() {
    this.initData();
  },
  mounted() {
    this.$bus.$on("find", content => {
      let data = {};
      let reg = new RegExp(/^""/);
      for (let key in content) {
        if (!reg.test(content[key]) && content[key] !== 0) {
          data[key] = content[key];
        }
      }
      // console.log(data);
      findmReport(data).then(res => {
        if (res.data.length === 0) {
          alert("该项目未提交中期报告或输入信息错误");
        } else {
          let tmp = null;
          console.log(res);
          this.tempList = [];
          res.data.forEach(item => {
            tmp = item;
            // console.log(tmp);
            tmp.userId = item.mreport.userId;
            tmp.projectId = item.mreport.projectId;
            tmp.mreport = 1;
            tmp.oneId = item.teacherName;
            this.collegeList.forEach(item => {
              if (item.name === tmp.collegeName) {
                tmp.collegeId = item.id;
              }
            });
            this.tempList.push(item);
          });
          console.log(this.tempList);
        }
      });
    });
  },
  methods: {
    //初始化数据
    initData() {
      this.userId = localStorage.getItem("USERID");
      request({
        url: "http://47.113.80.250:9003/project/select/" + this.userId,
        method: "get"
      }).then(res => {
        this.tableData = [];
        // console.log(res);
        res.data.forEach(item => {
          this.tableData.push(item);
        });
        // console.log(this.tableData);
        this.total = this.tableData.length;
        // console.log(this.count);
        this.tempList = this.tableData;
        console.log(this.tempList);

        return this.tempList;
      });
    },

    //获取所属二级学院名称
    GetCollegeName(row) {
      // console.log(row.collegeId);

      for (let i in this.collegeList) {
        if (row.collegeId === this.collegeList[i].id) {
          return this.collegeList[i].name;
        }
      }
    },
    //点击拓展按钮显示拓展表格
    handleMange(index, row) {
      // console.log(index);
      // console.log(row);
      request({
        url: "http://47.113.80.250:9003/user/select/" + row[0].userId,
        method: "get"
      }).then(res => {
        this.userInfo = [];
        this.userInfo = row;
        // console.log(res);
        this.userInfo.userCollege = res.data.college.collegeName;
        this.userInfo.userName = res.data.user.userName;
        this.userInfo.userPhone = res.data.user.phone;
        //将该项目中的数据整合到userInfo中
        // console.log(this.userInfo);
        // return this.tableData[0]
      });
    },
    //分页设置
    handleSizeChange(pageSize) {
      this.pageSize = pageSize;
      this.handleCurrentChange(this.currentPage);
    },
    handleCurrentChange(currentPage) {
      this.currentPage1 = currentPage;
      this.currentChangePage(this.tableData, currentPage);
      console.log(this.tableData);
    },
    currentChangePage(list, currentPage) {
      // console.log(list);
      // console.log(currentPage);

      let from = (currentPage - 1) * this.pageSize;
      let to = currentPage * this.pageSize;
      this.tempList = [];
      for (; from < to; from++) {
        if (list[from]) {
          this.tempList.push(list[from]);
        }
      }
      // console.log(this.tempList);
      // this.tableData = this.tempList
    },
    //认可中期报告
    mReportApproval(index, row) {
      // console.log("点击认可");
      // getmReport().then(res => {
      //   console.log(res);
      // });
      this.row = row;
      console.log(row);

      if (row.mreport === 1) {
        //操作
        request({
          url: "http://47.113.80.250:9003/report/select/" + row.projectId,
          method: "get"
        }).then(res => {
          this.reportId = res.data.mreport.reportId;
        });

        this.midReportNew.projectId = row.projectId;
        this.midReportNew.projectName = row.projectName;
        this.midReportNew.userId = row.userId;
        this.dialogFormVisibleNew = true;
        // console.log(row.projectId);

        // mReportApproval("teacher", {
        //   approval: 2,
        //   // comment: this.comment,
        //   comment: "老师认可测试",
        //   reportId: row.projectId
        // }).then(res => {
        //   console.log(res);
        // });
      } else if (row.mreport === 0) {
        alert("该报告未提交，无法审核！");
      }
    },
    pass() {
      // console.log(this.comment);

      mReportApproval("admin", {
        approval: 2,
        comment: this.comment,
        reportId: this.reportId
      }).then(res => {
        // console.log(res);
        this.comment = "";
        this.dialogFormVisibleNew = false;
      });
    },
    Back() {
      mReportApproval("admin", {
        approval: 3,
        comment: this.comment,
        reportId: this.reportId
      }).then(res => {
        this.comment = "";
        // console.log(res);
        this.dialogFormVisibleNew = false;
      });
    },
    //中期报告信息获取
    mReport(index, row) {
      console.log(row);
      request({
        url: "http://47.113.80.250:9003/report/select/" + row.projectId,
        method: "get"
      })
        .then(res => {
          // console.log(res);
          if (res.code === 200) {
            this.projectName = row.projectName;
            this.midReport = res.data.mreport;
            this.mfiles = res.data.mfiles;
            this.content = res.data.mreport.content;
            this.midReport.expertName = res.data.expertName;
            // console.log(this);

            return res.code;
          } else {
            alert(res.message);
          }
          // console.log(this.midReport);
        })
        .then(res => {
          // if(res.code == 200)
          if (res === 200) {
            this.dialogFormVisible = true;
          }
        });
    },

    //新建中期报告
    mReportNew(index, row) {
      console.log(row);
      if (row.mreport === 1) {
        alert("您已建立了中期报告，请查看并修改");
      } else if (row.mreport === 0) {
        this.midReportNew.projectId = row.projectId;
        this.midReportNew.projectName = row.projectName;
        this.midReportNew.userId = row.userId;
        this.dialogFormVisibleNew = true;
      }
    },
    //中期报告上传
    submitMreport() {
      request({
        url: "http://47.113.80.250:9003/report/student/insert",
        data: {
          content: this.midReportNew.content,
          projectId: this.midReportNew.projectId,
          userId: this.midReportNew.userId
        },
        method: "POST"
      }).then(res => {
        console.log(res);

        if (res.code === 200) {
          alert("新建成功！");
          this.dialogFormVisibleNew = false;
          this.$router.go(0);
        } else {
          alert(res.message);
        }
      });
    },

    //删除中期报告文件
    deleteFile(index, row) {
      let rowId = row.fid;
      request({
        url: "http://47.113.80.250:9003/report/student/file/delete",
        data: { fileId: rowId },
        method: "POST"
      }).then(res => {
        if (res.code === 200) {
          alert("删除成功");
          this.mfiles.splice(index, 1);
        } else {
          alert(res.message);
        }
      });
    },

    //修改中期报告简介
    changeContent() {
      request({
        url: "http://47.113.80.250:9003/report/student/update",
        data: { content: this.content, reportId: this.midReport.reportId },
        method: "POST"
      }).then(res => {
        alert("修改简介成功！");
        this.changeContentValue = false;
      });
    },

    //中期报告文件上传
    //(1)上传前检验
    beforeUpload(val) {
      console.log(val);
      let fileForm = new FormData();
      console.log(fileForm);

      fileForm.append("file", val.file);
      fileForm.append("reportId", this.midReport.reportId);
      this.mfileForm = fileForm;
      console.log(this.mfileNewForm);
      //下方显示文件名
      this.fileShow = true;
      this.mfileNewName = val.file.name;
      //限制文件大小不超过5m
      const isLt5M = val.file.size / 1024 / 1024 < 5;
      if (!isLt5M) {
        this.$message.error("上传文件大小不能超过 5MB!");
      }
      return isLt5M;
    },
    //(2)提交文件
    submitUpload() {
      console.log(this.mfileNewForm);
      if (this.fileShow !== true) {
        alert("请选择上传文件！");
      } else {
        request({
          url: "http://47.113.80.250:9003/report/student/file/insert/one",
          data: this.mfileForm,
          method: "POST"
        })
          .then(res => {
            if (res.code === 200) {
              alert(res.message);
              this.$router.go(0);
            } else {
              throw err;
            }
          })
          .catch(err => {
            alert(err);
          });
      }
    },
    //汇总中期报告
    sumReport() {
      request({
        url: "http://47.113.80.250:9003/report/admin/excel",
        method: "POST",
        responseType: "blob",
        headers: {
          "Content-Type": "application/json"
        }
      }).then(res => {
        const content = res;
        const blob = new Blob([content], { type: "application/ms-excel" });
        const fileName = "中期报告汇总表.xlsx"; //下载文件名称
        const elink = document.createElement("a");
        elink.download = fileName;
        //  elink.style.display = 'none';
        elink.href = URL.createObjectURL(blob);
        document.body.appendChild(elink);
        elink.click();
        URL.revokeObjectURL(elink.href); // 释放URL 对象
        document.body.removeChild(elink);
      });
    },
    //单个指派
    setExpert(row) {
      // console.log(row);
      request({
        url: "http://47.113.80.250:9003/report/select/" + row.projectId,
        method: "get"
      }).then(res => {
        console.log(res);
        if (res.code !== 200) {
          alert(res.message);
        } else if (res.code === 200) {
          // if (res.data.mreport.expert !== "") {
          //   alert("该项目已指派专家");
          // } else {
          this.setReportId = res.data.mreport.reportId;
          this.setExpertPid.push(this.setReportId);
          this.isShow = true;
          // }
        }

        // console.log(this.setReportId);
        // console.log(this.setExpertPid);
      });
      // console.log(this.setReportId);

      // console.log(this.setExpertPid);

      getAllExpert().then(res => {
        // console.log(res);
        res.data.forEach(item => {
          item.isSelect = false;
        });
        this.expert = res.data;
      });
    },
    cancel() {
      this.isShow = false;
    },

    //获取未指派中期文件还有专家名单
    appoint() {
      this.appointList = [];
      // console.log(this.tempList);
      this.userId = localStorage.getItem("USERID");
      request({
        url: "http://47.113.80.250:9003/report/admin/select/all/" + this.userId,
        method: "GET"
      }).then(res => {
        // console.log(res);
        this.getAllList = res.data;
        //  console.log(this.getAllList);
        this.getAllList.forEach(item => {
          if (item.expertName === null && item.submit === 1) {
            this.appointList.push(item);
          }
          console.log(this.appointList);

          this.appointedForm = true;
        });
      });
      // console.log(this.appointList);
      getAllExpert().then(res => {
        this.expertList = res.data;
        // console.log(this.expertList);
      });
    },
    //获取多选框选中的值
    handleSelectionChange(val) {
      //  this.chooseReport = val;
      val.forEach(item => {
        this.chooseReport.push(item.mreport.reportId);
      });
      console.log(this.chooseReport);
    },

    //处理专家按钮点击事件
    handleCommand(userId) {
      this.chooseExpert = userId;
    },
    //点击确认指派按钮事件
    submitAppoint() {
      if (this.chooseReport == "") {
        alert("请选择指派项目");
      } else if (this.chooseExpert == "") {
        alert("请选择指派专家");
      } else {
        request({
          url: "http://47.113.80.250:9003/report/admin/set/expert",
          data: {
            expert: this.chooseExpert,
            reportIds: this.chooseReport
          },
          method: "POST"
        }).then(res => {
          if (res.code === 200) {
            alert(res.message);
            this.$router.go(0);
          }
        });
        this.appointedForm = false;
      }
    },
    //中期报告文件下载
    download(row) {
      request({
        url: "http://47.113.80.250:9002/download",
        data: {
          fileUrl: row.furl,
          fileName: row.fname
        },
        method: "POST",
        responseType: "blob"
      }).then(res => {
        const content = res;
        const blob = new Blob([content]);
        const fileName = row.fname; //下载文件名称
        const elink = document.createElement("a");
        elink.download = fileName;
        //  elink.style.display = 'none';
        elink.href = URL.createObjectURL(blob);
        document.body.appendChild(elink);
        elink.click();
        URL.revokeObjectURL(elink.href); // 释放URL 对象
        document.body.removeChild(elink);
      });
    }
  }
};
</script>

<style scoped>
.eTable {
  position: absolute;
  top: 160px;
  left: 380px;
  width: 860px !important;
}

.table-expand {
  font-size: 0;
}

.table-expand label {
  width: 90px;
  color: #99a9bf;
}
.table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  line-height: 20px;
  width: 50%;
}
.el-pagination {
  text-align: center;
  margin-left: -50px;
  margin-top: 30px;
}
.content-view {
  /* position: fixed; */
  /* margin-top: 80px; */
  padding-top: 100px;
  padding-left: 50px;
  overflow: auto;
}
.project_form {
  margin-top: 0px;
  margin-left: 0px;
}
.table-expand {
  margin: 0 auto;
}
.el-table__expanded-cell form {
  margin-top: -10px;
  width: 100%;
}
.table-expand el-button {
  margin: 0 auto;
}
.el-dialog__body {
  margin: -5px auto 0;
}
/* .el-form-item {
      margin-left: 15px;
    } */

.el-dialog__title {
  font-size: 16px;
  font-weight: 600;
  color: cadetblue;
}
.midReport {
  margin: 0 auto;
}
.mReportForm {
  margin: 0;
  margin-top: -18px;
  width: 100%;
}
.v-modal {
  position: relative;
}
.el-form-item {
  margin-bottom: 5px;
}
.el-form-item__label,
.el-form-item__content {
  line-height: 30px;
}
.midDisscuss {
  width: 50%;
  float: left;
}
.el-button {
  vertical-align: middle;
}
.changeContent {
  color: skyblue;
  font-size: 14px;
  margin-left: 10px;
  text-decoration: none;
}
.contentForm {
  margin: 0;
}
.contentForm .el-button {
  float: right;
  margin-top: 3px;
}
.el-col-20 {
  width: 74.1%;
}
.block .el-pagination {
  margin-top: 10px;
  margin-bottom: 10px;
}

.appoint span {
  color: darkslategray;
}
.sumApp {
  position: absolute;
  right: 145px;
  bottom: 10px;
}
</style>
