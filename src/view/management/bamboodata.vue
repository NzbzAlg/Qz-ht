<template>
  <div>
    <sx></sx>
    <!-- 筛选 -->
    <h2 style="text-align: left; margin-top: 0;">清竹数据({{zhanghu}})</h2>
    <!-- 数据统计 -->
    <el-row :class="$style.f_row">
      <el-col :span="3" :class="$style.f_da">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{vkt}}</span>
          <p>余额(VKT)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{dataincome}}</span>
          <p>收益(数据收益)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{taskincome}}</span>
          <p>收益(任务收益)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{dataorder}}</span>
          <p>支出(数据订购)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{datarepo}}</span>
          <p>支出(数据回购)</p>
        </el-card>
      </el-col>
    </el-row>
    <div :class="$style.f_sx">
      <el-select v-model="value" clearable placeholder="请选择" :class="$style.f_sxk">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-date-picker v-model="value1" type="date" placeholder="选择日期" value-format="yyyy-MM-dd" :class="$style.f_sxk1"></el-date-picker>
      <el-button type="primary" size="medium" icon="el-icon-search" @click="search">搜索</el-button>
    </div>
    <!-- 表格 -->
    <div>
      <el-table :data="tableData" border style="width: 100%">
        <el-table-column prop="create_time" label="日期"></el-table-column>
        <el-table-column prop="account" label="账户"></el-table-column>
        <el-table-column prop="name" label="类型">
          <template slot-scope="scope">
            <span v-if="scope.row.type===1">平台充值</span>
            <span v-if="scope.row.type===2">平台提现</span>
            <span v-if="scope.row.type===3">自充值</span>
            <span v-if="scope.row.type===4">自提现</span>
            <span v-if="scope.row.type===5">数据收益</span>
            <span v-if="scope.row.type===6">任务奖励</span>
            <span v-if="scope.row.type===7">订购数据</span>
            <span v-if="scope.row.type===8">数据画像</span>
            <span v-if="scope.row.type===9">固定画像</span>
          </template>
        </el-table-column>
        <el-table-column prop="address" label="单位">
          <template slot-scope="scope">
            <span v-if="scope.row.currency===1">VKT</span>
            <span v-if="scope.row.currency===2">CNY</span>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <!-- 分页 -->
    <div>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage4"
        :page-sizes="[10, 20, 30, 40]"
        :page-size="100"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>
  </div>
</template>

<script>
import sx from '../components/sxbtn'
export default {
  components: {
    sx
  },
  data () {
    return {
      zhanghu:'',
      value: '',
      value1: '',
      options: [
        {
          value: 0,
          label: '全部'
        },
        {
          value: 1,
          label: '数据服务'
        },
        {
          value: 2,
          label: '数据收益'
        },
        {
          value: 3,
          label: '任务奖励'
        },
        {
          value: 4,
          label: '数据回购'
        }
      ],
      tableData: [],
      vkt: '',
      cny: '',
      dataincome: "",
      taskincome: '',
      dataorder: '',
      datarepo: "",
      currentPage4: 1,
      total: null,
      sizes: 10,
      pages: 0,
    }
  },
  mounted () {
    this.getCount();
    this.getList();
  },
  methods: {
    getCount () {
      this.$http.get(`modules/account/qzDataCount`).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.zhanghu = data.account
          this.vkt = data.vkt
          this.dataincome = data.income
          this.taskincome = data.taskIncome
          this.dataorder = data.data
          this.datarepo = data.reward
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    getList () {
      this.$http.get(`modules/account/qzlist`, {
        params: {
          size: this.sizes,
          page: this.pages,
          tab: 0
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          console.log(data)
          this.tableData = data.content
          this.total = data.total
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    search(){
      this.$http.get(`modules/account/qzlist`, {
        params: {
          size: this.sizes,
          page: this.pages,
          tab: this.value,
          values:this.value1
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          console.log(data)
          this.tableData = data.content
          this.total = data.total
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    handleSizeChange (val) {
      this.sizes = val
      this.$http.get(`modules/account/qzlist`, {
        params: {
          size: val,
          page: this.pages,
          tab: 0
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          console.log(data)
          this.tableData = data.content
          this.total = data.total
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    handleCurrentChange (val) {
      this.pages = val-1
      this.$http.get(`modules/account/qzlist`, {
        params: {
          size: this.sizes,
          page:  val-1,
          tab: 0
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          console.log(data)
          this.tableData = data.content
          this.total = data.total
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
  }
}
</script>

<style lang='scss' module>
.f_sx {
  height: 40px;
  margin-bottom: 10px;
  .f_sxk,
  .f_sxk1 {
    width: calc(100% - 75%);
    float: left;
  }
  .f_sxk1 {
    margin-left: 20px;
  }
}

.f_row {
  margin-bottom: 20px;
  .f_da,
  .f_sheng {
    margin: 0 24px;
  }
}
.f_sl {
  margin-top: 0;
  font-family: "MicrosoftYaHei", "微软雅黑";
  font-size: 28px;
}
</style>
