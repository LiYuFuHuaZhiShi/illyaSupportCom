<template>
  <div>
    <el-dialog title="提示" :visible.sync="dialogVisible" width="500px">
      <el-upload class :show-file-list="false" drag action :http-request="uploadfile" multiple>
        <img v-if="loadcover" :src="loadcover" style="width: 100%" />
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">
          将文件拖到此处，或
          <em>点击上传</em>
        </div>
      </el-upload>

      <el-radio-group v-model="upload_category">
        <el-radio-button label="1">原创</el-radio-button>
        <el-radio-button label="2">Pixiv</el-radio-button>
        <el-radio-button label="3">Cospaly</el-radio-button>
      </el-radio-group>

      <el-input placeholder="请输入内容" v-model="upload_title">
        <template slot="prepend">标题</template>
      </el-input>

      <el-input
        type="textarea"
        placeholder="请输入内容"
        v-model="upload_info"
        maxlength="30"
        show-word-limit
      ></el-input>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="upload()">确 定</el-button>
      </span>
    </el-dialog>
    <div class="Common page-width content">
      <el-row>
        <el-button @click="dialogVisible = true">上传图片</el-button>
      </el-row>

      <el-row class="item-content">
        <div v-for="(item, index) in list" :key="index" class="item">
          
          <div class="rb" @click="del(item.id)">删除</div>
          <div class="l">
            <img :src="item.cover" style="width: 100%" />
          </div>
          <div class="r">
            <div class="lc">
              <div class="title">{{item.title}} - (
                <span v-if="item.verify == 1">以通过审核</span>
                <span v-if="item.verify == 2">等待审核中</span>
                <span v-if="item.verify == 3">已被退回</span>
                )</div>
              <div class="date">上传时间: {{item.create_time}}</div>
              <div class="info">介绍: {{item.info}}</div>
            </div>
          </div>
          
        </div>
      </el-row>

    </div>
    <Pagination
      :total="totalItem"
      :page.sync="currentPage"
      @pagination="getList()"
      :limit.sync="pageSize"
    />
  </div>
</template>

<script>
import Pagination from "@/components/Pagination.vue";
export default {
  name: "photos",
  data() {
    return {
      category: 0,
      currentPage: 1, // 当前页码
      totalItem: 0, // 总条目
      totalPage: 0, // 总页数
      pageSize: 10, // 每页多少条
      list: [],

      upload_file: "",
      upload_title: "",
      upload_info: "",
      upload_category: 0,
      dialogVisible: false,
      loadcover: ""
    };
  },
  components: {
    Pagination
  },
  methods: {
    del(val){
      this.$http
        .PhotoChange({
          id:val,
          set:1
        })
        .then(response => {
          if (response.code == 200) {
            this.getList();
          }
        })
        .catch(error => {
          console.log("error", error);
        });
    },
    uploadfile(file) {
      const formData = new FormData();
      formData.append("file", file.file);
      formData.append("uploadKey", "photo");

      console.log(file);
      this.$http.upload(formData).then(res => {
        console.log(res);
        console.log("上传成功");
        if (res.code === 200) {
          // console.log('res.data:',res.data)
          this.upload_file = res.data.filename;
          this.loadcover = res.data.lodpath;
        }
      });
    },
    upload() {
      this.$http
        .PhotoUp({
          file: this.upload_file,
          title: this.upload_title,
          info: this.upload_info,
          category: this.upload_category
        })
        .then(response => {
          if (response.code == 200) {
            console.log(response.msg);
            this.dialogVisible = false;
            this.upload_file = ''
            this.upload_title = ''
            this.upload_info = ''
            this.upload_category = 0
            this.loadcover = ''
            this.getList();
          }
        })
        .catch(error => {
          console.log("error", error);
        });
    },
    getList() {
      this.$http
        .PhotoList({
          category: this.category,
          sfilter: 1,
          pages: this.currentPage,
          userid: this.$authUser.getUser().userID
        })
        .then(response => {
          if (response.code == 200) {
            this.list = response.data.result;
            this.currentPage = response.data.page;
            this.totalPage = response.data.pages;
            this.totalItem = response.data.count;
          }
        })
        .catch(error => {
          console.log("error", error);
        });
    }
  },
  created() {
    let user = this.$authUser.getUserToken();
    if (user == null) {
      this.$router.push({ name: "login" });
    }
    this.getList();
  }
};
</script>
<style lang="scss" scoped>
.content {
  margin-top: 25px;
}
.item-content{ margin-top: 15px;}
.item {
  position: relative;
  height: 200px;
  border: 1px solid #f4f4f4;
  padding: 15px;
  margin-bottom: 15px;
  display: flow-root;
  .rb{width: 80px;background-color: #f4f4f4;height: 100%;position: absolute;right: 0;top: 0;bottom: 0;text-align: center;text-align-last: center;font-size: 14px;line-height: 230px;}
  .rb:hover{background-color: #acacac;color: #fff;}
  .l{width: 200px;float: left;height: 200px;overflow: hidden;}
  .r{
    .lc{
      width: calc(100% - 80px);
      .title{width: 100%;font-size: 18px;}
    .info{width: 100%;font-size: 12px;margin-top: 30px;height: 80px;}
    .date{width: 100%;font-size: 12px;margin-top: 5px;}
    }
    width: calc(100% - 215px);float: right;
    }
}
</style>