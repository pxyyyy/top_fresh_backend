<style lang="less">
@import "../../../styles/common.less";
</style>
<template>
    <div>
        <Row>
            <Col span="16">
                <Row>
                    <Card>
                        <div style="border-bottom: 1px solid #ccc; margin-top: 20px">
                            <h1>商品锦集活动设置</h1>
                        </div>
                        <div style="margin-bottom: 20px;margin-top:10px">
                            <span>活动名称：</span>
                            <Input v-model="articleTitle"  icon="android-list"  style="width:300px"/>
                        </div>
                        <div class="demo-upload-list" v-for="item in uploadList">
                            <template v-if="item.status === 'finished'">
                                <img :src="item.imgUrl">
                                <div class="demo-upload-list-cover">
                                    <Icon type="ios-eye-outline" @click.native="handleView(item.imgUrl)"></Icon>
                                    <Icon type="ios-trash-outline" @click.native="handleRemove(item)"></Icon>
                                </div>
                            </template>
                            <template v-else>
                                <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
                            </template>
                        </div>
                         <Upload ref="upload" :show-upload-list="false" :default-file-list="defaultList" :on-success="handleSuccess" :format="['jpg','jpeg','png']" :max-size="2048" :on-format-error="handleFormatError" :on-exceeded-size="handleMaxSize" :before-upload="handleBeforeUpload" multiple type="drag"  :action="url" style="display: inline-block;width:58px;">
                          <div style="width: 58px;height:58px;line-height: 58px;">
                            <Icon type="camera" size="20"></Icon>
                          </div>
						            </Upload>
                        <Modal title="View Image" v-model="visible">
                            <img :src=" imgName" style="width: 100%">
                        </Modal>
                       
                    </Card>
                    <Card style="margin-top: 10px;">
                      <p slot="title" style="height:35px">
                      <Icon type="ios-film-outline"></Icon>
                        商品集锦活动详情
                        <span class="publish-button" style="float:right;margin-right:25px">
                          <Button type="primary" @click="show = true">添加商品</Button>
                        </span>
                      </p>
                        <Table stripe border :columns="tableColumns1" :data="tableData1"></Table>
                        <div style="margin: 10px;overflow: hidden">
                            <div style="float: right;">
                                <Page show-elevator show-sizer @on-page-size-change="changePageSize" :total="count" :current="1" @on-change="changePage"></Page>
                            </div>
                        </div>
                    </Card>
                </Row>
            </Col>
            <Col span="8" class="padding-left-10">
                <Row>
                    <Card>
                        <h4>活动时间设置</h4>
                        <Card style="margin-top: 20px">
                            <div style="margin: 20px 0">
                                <span>活动开始时间</span>
                                <DatePicker type="date" placement="bottom-end" placeholder="请选择" style="width: 200px; margin-left: 10px" @on-change="selectTime" v-model="start"></DatePicker>
                                
                            </div>
                            <div style="margin: 20px 0">
                                <span>活动结束时间</span>
                                <DatePicker type="date" placement="bottom-end" placeholder="请选择" style="width: 200px; margin-left: 10px" @on-change="selectTime1" v-model="end"></DatePicker>
                                
                            </div>
                            <div style="text-align: center" v-if="isAdd" >
                                <Button type="primary" style="width: 30%" @click="add">添加</Button>
                            </div>
                            <div style="text-align: center" v-else>
                                <Button type="primary" style="width: 30%" @click="save">保存</Button>
                            </div>
                        </Card>
                    </Card>
                </Row>
                <Row style="margin-top: 10px">
                </Row>
            </Col>
        </Row>
        <Modal
        v-model="show"
        title="Common Modal dialog box title"
        @on-ok="ok"
        @on-cancel="cancel">
        <p style="margin-bottom:30px">
          <span>请选择商品</span>
          <Select  clearable style="width:200px" @on-change="product">
            <Option v-for="item in typeList" :value="item.value" :key="item.value">{{ item.label }}</Option>
          </Select>
           <Select v-model="productId" clearable style="width:200px">
            <Option v-for="item in productList" :value="item.productId" :key="item.productId">{{ item.productName }}</Option>
          </Select>
        </p>
        <!-- <p>
          <span>设置商品数量</span>
          <Input v-model="productNum"  icon="android-list"  style="width:300px"/>
        </p> -->
    </Modal>
    </div>
</template>
<script>
import Cookies from "js-cookie";
import active from "../service/active.js";
export default {
  mixins: [active],
  data() {
    return {
      isAdd: true,
      show: false,
      articleTitle: "",
      value: "1580元",
      model1: "",
      model2: "",
      url: "",
      active: "",
      start: "",
      end: "",
      productList: [],
      productId: "",
      productNum: "",
      defaultList: [],
      typeList: [
        {
          value: "1",
          label: "礼卡"
        },
        {
          value: "2",
          label: "现货"
        }
      ],
      pageSize: 10,
      pageNum: 1,
      count: 10,
      imgName: "",
      acCreattime: "",
      acEndtime: "",
      visible: false,
      uploadList: [],
      tableData1: [],
      tableColumns1: [
        {
          title: "序号",
          key: "productId",
          width: 80,
          align: "center"
        },
        {
          title: "商品名称",
          key: "productName",
          align: "center"
        },
        {
          title: "价格",
          key: "productPrice",
          align: "center"
        }
      ]
    };
  },
  methods: {
    selectTime1(val) {
      this.acEndtime = val;
    },
    selectTime(val) {
      this.acCreattime = val;
    },
    save() {
      var that = this;
      var token = Cookies.get("token");
      var staffId = Cookies.get("staffId");
      var params = {
        token,
        staffId,
        acTitle: this.articleTitle,
        acIcon: this.defaultList[0].imgUrl,
        acCreattime: this.acCreattime,
        acEndtime: this.acEndtime,
        acType: 3,
        acId: this.activeId
      };
      this.updateActivity(params).then(res => {
        if (res.code == 100000) {
          this.$Message.success({
            content: "修改活动成功",
            onClose: function() {
              that.mockTableData1();
            }
          });
        } else {
          this.$Message.error(res.message);
        }
      });
    },
    ok() {
      var that = this;
      var token = Cookies.get("token");
      var staffId = Cookies.get("staffId");
      var params = {
        token,
        staffId,
        activityId: this.activeId,
        productId: this.productId,
        productNum: this.productNum
      };
      this.updateProBack(params).then(res => {
        if (res.code == 100000) {
          this.$Message.success({
            content: "添加成功",
            onClose: function() {
              that.mockTableData1();
            }
          });
        } else {
          this.$Message.error(res, message);
        }
      });
    },
    cancel() {
      this.$Message.info("Clicked cancel");
    },
    product(val) {
      var token = Cookies.get("token");
      var staffId = Cookies.get("staffId");
      var params = {
        token,
        staffId,
        pageNum: 1,
        pageSize: 1000000,
        productPtype: val,
        productIsuse: 1
      };
      this.getProduct(params).then(res => {
        if (res.code == 100000) {
          this.productList = res.data;
        }
      });
    },
    add() {
      var that = this;
      var token = Cookies.get("token");
      var staffId = Cookies.get("staffId");
      var params = {
        token,
        staffId,
        acTitle: this.articleTitle,
        acIcon: this.defaultList[0].imgUrl,
        acCreattime: this.acCreattime,
        acEndtime: this.acEndtime,
        acType: 3
      };
      this.addActivity(params).then(res => {
        if (res.code == 100000) {
          this.$Message.success({
            content: "添加活动成功",
            onClose: function() {
              that.activeId = res.data;
              that.mockTableData1();
            }
          });
        } else {
          this.$Message.error(res.message);
        }
      });
    },
    mockTableData1() {
      var token = Cookies.get("token");
      var staffId = Cookies.get("staffId");
      var params = {
        token,
        staffId,
        activityId: this.activeId,
        pageNum: this.pageNum,
        pageSize: this.pageSize
      };
      this.selectListBack(params).then(res => {
        if (res.code == 100000) {
          this.tableData1 = res.data;
          this.count = res.count;
        }
      });
    },
    changePageSize(pageSize) {
      this.pageSize = pageSize;
      this.mockTableData1();
    },
    changePage(pageNum) {
      this.pageNum = pageNum;
      this.mockTableData1();
    },
    // 图片上传
    handleView(name) {
      this.imgName = name;
      this.visible = true;
    },
    handleRemove(file) {
      const fileList = this.$refs.upload.fileList;
      this.$refs.upload.fileList.splice(fileList.indexOf(file), 1);
    },
    handleSuccess(res, file) {
      this.defaultList = [];
      this.defaultList.push({ imgUrl: file.response.data, imgName: file.name });
      this.$nextTick(() => {
        this.uploadList = this.$refs.upload.fileList;
      });
    },
    handleFormatError(file) {
      this.$Notice.warning({
        title: "The file format is incorrect",
        desc:
          "File format of " +
          file.name +
          " is incorrect, please select jpg or png."
      });
    },
    handleMaxSize(file) {
      this.$Notice.warning({
        title: "Exceeding file size limit",
        desc: "File  " + file.name + " is too large, no more than 2M."
      });
    },
    handleBeforeUpload() {
      const check = this.uploadList.length < 1;
      if (!check) {
        this.$Notice.warning({
          title: "Up to one pictures can be uploaded."
        });
      }
      return check;
    }
  },
  mounted() {
    var token = Cookies.get("token");
    var staffId = Cookies.get("staffId");
    this.url =
      "http://39.107.126.201:8080/fresh_show//User/uploadAll?token=" +
      token +
      "&staffId=" +
      staffId +
      "&type=1";
    this.uploadList = this.$refs.upload.fileList;
    var type = this.$route.params.activeId;
    if (type != "add") {
      this.isAdd = false;
      this.activeId = type;
      this.mockTableData1();
      var params = {
        token,
        staffId,
        acId: type
      };
      this.selectByPrimaryKey(params).then(res => {
        if (res.code == 100000) {
          this.articleTitle = res.data[0].acTitle;
          this.defaultList.push({
            imgUrl: res.data[0].acIcon,
            imgName: res.data[0].acTitle
          });
          this.$nextTick(() => {
            this.uploadList = this.$refs.upload.fileList;
          });
          this.start = res.data[0].acCreattime;
          this.end = res.data[0].acEndtime;
        }
      });
    }
  }
};
</script>
<style>
.demo-upload-list {
  display: inline-block;
  width: 60px;
  height: 60px;
  text-align: center;
  line-height: 60px;
  border: 1px solid transparent;
  border-radius: 4px;
  overflow: hidden;
  background: #fff;
  position: relative;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
  margin-right: 4px;
}
.demo-upload-list img {
  width: 100%;
  height: 100%;
}
.demo-upload-list-cover {
  display: none;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.6);
}
.demo-upload-list:hover .demo-upload-list-cover {
  display: block;
}
.demo-upload-list-cover i {
  color: #fff;
  font-size: 20px;
  cursor: pointer;
  margin: 0 2px;
}
</style>
