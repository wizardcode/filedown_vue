<template>
  <div class="content">
    <div class="btn_class">
      <el-button type="primary" @click="dialogFormVisible = true">添加任务</el-button>
      <el-button type="success" @click="allrun">全部开始</el-button>
      <el-button type="info" @click="loaddata">获取结果</el-button>
    </div>
    <el-table :data="tableData" border align:center style="width: 100%;">
      <el-table-column type="index" header-align="center">
      </el-table-column>
      <el-table-column label="URL" width="300" header-align="center" prop="1">
      </el-table-column>
      <el-table-column label="文件路径" width="300" header-align="center" prop="3">
      </el-table-column>
      <el-table-column label="状态" width="100" header-align="center" prop="2" :formatter="showstatus">
      </el-table-column>
      <el-table-column label="操作" header-align="center">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleStart(scope.$index, scope.row)" v-bind:disabled="scope.row[2]==3 || scope.row[2]==2 || scope.row[2]==4">开始</el-button>
          <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="添加文件地址" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="文件地址">
          <el-input type="textarea" v-model="form.url" :rows="5"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addfile">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
export default {
  data() {
    return {
      tableData: [],
      dialogFormVisible: false,
      form: {
        id: "",
        url: ""
      }
    };
  },
  methods: {
    handleStart(index, row) {
      Vue.set(this.tableData[index], 2, 2);
      axios({
        url: "/download",
        method: "post",
        data: {
          id: this.tableData[index][0]
        }
      }).then(
        res => {
          this.loaddata();
        },
        error => {
          console.log("任务进行中...");
        }
      );
    },
    showstatus(row, column) {
      if (row[2] == 1) {
        return "下载失败";
      } else if (row[2] == 0) {
        return "未开始";
      } else if (row[2] == 2) {
        return "下载中...";
      } else if (row[2] == 3) {
        return "已完成";
      } else if (row[2] == 4) {
        return "禁止下载";
      } else {
        return "未开始";
      }
    },
    handleDelete(index, row) {
      axios({
        url: "/deletefile",
        method: "post",
        data: {
          id: this.tableData[index][0]
        }
      }).then(res => {
        this.loaddata();
      });
    },
    addfile() {
      this.dialogFormVisible = false;
      if (this.form.url == "") {
        return;
      }
      axios({
        url: "/addfile",
        method: "post",
        data: {
          url: this.form.url
        }
      }).then(res => {
        this.form.url = "";
        this.loaddata();
      });
    },
    allrun() {
      for (var i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i][2] == 0) {
          Vue.set(this.tableData[i], 2, 2);
        }
      }
      axios.get("/download").then(
        res => {
          this.loaddata();
        },
        error => {
          console.log("任务进行中...");
        }
      );
    },
    loaddata() {
      axios.get("/getfile").then(res => {
        this.tableData = res.data;
      });
    }
  },
  mounted() {
    this.loaddata();
  }
};
</script>
<style scoped>
.content {
  width: 1000px;
  margin: 0 auto;
  text-align: center;
}
.btn_class {
  float: right;
  margin-bottom: 20px;
}
</style>

