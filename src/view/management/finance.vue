<template>
  <div>
    <sx></sx>

    <!-- 数据统计 -->
    <el-row :class="$style.f_row">
      <el-col :span="3" :class="$style.f_da" style="margin:0px 10px;width:22%;">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{specialVkt}}</span>
          <p>大客户余额(VKT)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng" style="margin:0px 10px;width:22%;">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{specialCny}}</span>
          <p>大客户余额(CNY)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng" style="margin:0px 10px;width:22%;">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{merchantVktBalance}}</span>
          <p>代理商余额(VKT)</p>
        </el-card>
      </el-col>
      <el-col :span="3" :class="$style.f_sheng" style="width:22%;margin:0 10px">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{incomeVKT}}</span>
          <p>收益(VKT)</p>
        </el-card>
      </el-col>
    </el-row>
    <!-- 搜索 -->
    <div :class="$style.f_search">
      <el-input placeholder="请输入客户名称" v-model="name" :class="$style.f_ipt" clearable></el-input>
      <el-select v-model="proxyType" clearable placeholder="请选择" :class="$style.f_ipt1">
        <el-option
          v-for="item in options1"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>
      <el-button type="primary" size="medium" @click="search" icon="el-icon-search">搜索</el-button>
    </div>
    <!-- 表格 -->
    <div>
      <el-table :data="tableData" border style="width: 100%" v-loading="loading">
        <el-table-column label="客户名称">
          <template slot-scope="scope">
            <span :class="$style.f_bh" @click="bh(scope.row)">{{ scope.row.name}}</span>
          </template>
        </el-table-column>
        <el-table-column label="客户类型">
          <template slot-scope="scope">
            <span v-if="scope.row.proxy_type===1">省级运营中心</span>
            <span v-if="scope.row.proxy_type===2">市级运营中心</span>
            <span v-if="scope.row.proxy_type===3">市级一般代理商</span>
            <span v-if="scope.row.proxy_type===4">大客户</span>
            <span v-if="scope.row.proxy_type===5">清竹数据</span>
          </template>
        </el-table-column>
        <el-table-column label="区域">
          <template slot-scope="scope">
            <span>{{scope.row.province}}\{{scope.row.city}}</span>
          </template>
        </el-table-column>
        <el-table-column prop="ddid" label="CNY余额">
          <template slot-scope="scope">
            <span>{{scope.row.cnyAmount}}</span>
          </template>
        </el-table-column>
        <el-table-column prop="name" label="VKT余额">
          <template slot-scope="scope">
            <span>{{scope.row.vktAmount}}</span>
          </template>
          </el-table-column>
        <el-table-column prop="income" label="收益"></el-table-column>
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
import { mapActions } from 'vuex'
export default {
  components: {
    sx
  },
  data () {
    return {
      dialogVisible: false,
      specialVkt: '',
      specialCny: '',
      merchantVktBalance: '',
      incomeVKT: '',
      input: '',
      input1: '',
      input2: '',
      value1: '',
      value2: '',
      loading:true,
      formkhfull: {},
      options: [
        {
          value: '选项1',
          label: '未开始'
        }, {
          value: '选项2',
          label: '进行中'
        }, {
          value: '选项3',
          label: '结束'
        }, {
          value: '选项4',
          label: '取消'
        },
      ],
      options1: [
        {
          value: 1,
          label: '省级运营中心'
        },
        {
          value: 2,
          label: "市级运营中心"
        },
        {
          value: 3,
          label: "市级一般代理商"
        },
        {
          value: 4,
          label: "大客户"
        },
        {
          value: 5,
          label: "清竹数据"
        }
      ],
      value: '',
      tableData: [],
      currentPage4: 1,
      total: null,
      sizes: 10,
      name: '',
      proxyType: '',
      pages: 0,
    }
  },
  mounted () {
    this.getCount()
    this.getList()
  },
  methods: {
    ...mapActions([
      'getSearch'
    ]),
    getCount () {
      this.$http.get(`modules/financeAfter/customerFinance`).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.specialVkt = data.specialVkt
          this.specialCny = data.specialCny
          this.merchantVktBalance = data.merchantVktBalance
          this.incomeVKT = data.incomeVKT
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    getList () {
      let info = {
        size: this.sizes,
        page: this.pages
      }
      this.$http.post(`modules/financeAfter/customerList`, info).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.loading = false
          this.tableData = data.content
          this.total = data.total
          let province = ''
          let city= ''
          this.tableData.forEach(item=>{
           city =  item.city 
          })
          // this.getSearch().then(res => {
          //   res.forEach(item => {
          //     // console.log(item)
          //     this.tableData.forEach(item1 => {
          //       if (item1.province === item.id) {
          //         province = item.name
          //       }
          //     })
          //   })
          // })
          // this.getSearch(city).then(res => {
          //   res.forEach(item => {
          //     console.log(item)
          //   })
          // })
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    search () {
      let info = {
        'name': this.name,
        'proxyType': this.proxyType,
        'size': this.size
      }
      this.$http.post(`modules/financeAfter/customerList`, info).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
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
    chongzhi1 () {
      this.dialogVisible = true
    },
    handleClose () {
      this.dialogVisible = false
    },
    bh (row) {
      console.log(row)
      this.obj = {
        title: '客户财务',
        name: '/index/caiwuxiangqing.vue',
        name: row.name
      }
      this.$store.commit('mypush', this.obj)
      this.$router.push({ path: '/index/caiwuxiangqing.vue', query: { 
          name: row.name,
          province:row.province,
          city:row.city,
          proxy_type:row.proxy_type,
          vktAmount:row.vktAmount ,
          cnyAmount:row.cnyAmount ,
        }
      })
    },
    handleSizeChange (val) {
      this.sizes = val
      let info = {
        size: val,
        page: this.pages
      }
      this.$http.post(`modules/financeAfter/customerList`, info).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
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
      this.pages = val
      let info = {
        size: this.sizes,
        page: val - 1
      }
      this.$http.post(`modules/financeAfter/customerList`, info).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
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
.f_search {
  height: 40px;
  margin-bottom: 10px;
  .f_ipt,
  .f_ipt1 {
    width: calc(100% - 87%);
    float: left;
  }
  .f_ipt1 {
    margin-left: 20px;
  }
}
.f_sx {
  height: 40px;
  .f_sxk,
  .f_sxk1 {
    width: calc(100% - 80%);
    float: left;
  }
  .f_sxk1 {
    margin-left: 20px;
  }
}
.f_row1 {
  float: left;
  margin-top: 20px;
  margin-bottom: 10px;
}
h3 {
  margin-top: 0;
}
.f_bh {
  cursor: pointer;
}
.f_row {
  margin-bottom: 20px;
  .f_da,
  .f_sheng {
    margin: 0px 67px;
    width: 13%;
  }
}
.f_sl {
  margin-top: 0;
  font-family: "MicrosoftYaHei", "微软雅黑";
  font-size: 28px;
}
</style>
