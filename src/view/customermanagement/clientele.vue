<template>
  <div>
    <sx></sx>
    <!-- 客户统计 -->
    <el-row :class="$style.f_row">
      <el-col :span="4" :style="$style.f_da">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{all}}</span>
          <p>全部</p>
        </el-card>
      </el-col>
      <el-col :span="4" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{bigkh}}</span>
          <p>大客户</p>
        </el-card>
      </el-col>
      <el-col :span="4" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{shengjyyzx}}</span>
          <p>省级运营中心</p>
        </el-card>
      </el-col>
      <el-col :span="4" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{sjyyzx}}</span>
          <p>市级运营中心</p>
        </el-card>
      </el-col>
      <el-col :span="4" :class="$style.f_sheng">
        <el-card shadow="hover">
          <span :class="$style.f_sl">{{ybdl}}</span>
          <p>市级一般代理商</p>
        </el-card>
      </el-col>
    </el-row>
    <el-input placeholder="请输入关键字" v-model="name" :class="$style.name" clearable style="width:20%"></el-input>
    <el-select v-model="value" placeholder="请选择" :class="$style.office">
      <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id"></el-option>
    </el-select>
    <el-button type="primary" icon="el-icon-search" @click="search">搜索</el-button>
    <el-button type="primary" @click='client'>客户创建</el-button>

    <!-- 表格 -->
    <div :class="$style.table">
      <el-table ref="singleTable" :data="tableData" highlight-current-row style="width: 100%" v-loading="loading">
        <el-table-column type="index" label="序号" width="150"></el-table-column>
        <el-table-column label="客户名称" sortable>
          <template slot-scope="scope">
            <div @click="khname(scope.$index, scope.row)" :class="$style.f_khnam">{{scope.row.name}}</div>
          </template>
        </el-table-column>
        <el-table-column property="company_name" label="企业名称" sortable></el-table-column>
        <el-table-column property="stamp" label="类型" sortable>
          <template slot-scope="scope">
            <span v-if="scope.row.proxy_type===1">省级运营中心</span>
            <span v-if="scope.row.proxy_type===2">市级运营中心</span>
            <span v-if="scope.row.proxy_type===3">市级一般代理商</span>
            <span v-if="scope.row.proxy_type===4">大客户</span>
            <span v-if="scope.row.proxy_type===5">清竹数据</span>
            <span v-if="scope.row.proxy_type===6">合资公司</span>
          </template>
        </el-table-column>
        <el-table-column property="region" label="区域" sortable></el-table-column>
        <el-table-column label="操作" width="400" style="text-align:center">
          <template slot-scope="scope">
            <el-button size="mini" type="success" @click="handleDelete(scope.$index, scope.row)">重置密码</el-button>
            <el-button size="mini" type="primary" @click="khname(scope.$index, scope.row)">编辑</el-button>
            <el-button size="mini" type="warning" @click="empower(scope.$index, scope.row)">赋权</el-button>
            <el-button size="mini" type="warning" v-if="scope.row.status===1" @click="Disable(scope.$index, scope.row)">停用</el-button>
            <el-button size="mini" type="primary" v-if="scope.row.status===2" @click="Enable(scope.$index, scope.row)">启用</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage4"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="100"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
    <!-- 详情 -->
    <el-dialog title="提示" :visible.sync="full" width="30%" :before-close="handleClose">
      <el-form :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
        <el-form-item label="客户名称">
          <el-input v-model="formLabelAlign.name"></el-input>
        </el-form-item>
        <el-form-item label="企业名称">
          <el-input v-model="formLabelAlign.qyname"></el-input>
        </el-form-item>
        <el-form-item label="类型">
          <el-input v-model="formLabelAlign.stamp"></el-input>
        </el-form-item>
        <el-form-item label="区域">
          <el-input v-model="formLabelAlign.region"></el-input>
        </el-form-item>
      </el-form>
      <span>
        <el-button @click="full = false">取 消</el-button>
        <el-button type="primary" @click="full = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 客户详情 -->
    <el-dialog
      title="客户详情"
      :visible.sync="khfull"
      width="40%"
      :before-close="handleClose1"
      :class="$style.f_khxq"
    >
      <el-form
        :label-position="labelPosition"
        label-width="100px"
        :model="formkhfull"
        :class="$style.f_from"
      >
        <el-divider content-position="left" style="margin:16px 0;">基本信息</el-divider>
        <el-row>
          <el-col :span="12" :class="$style.f_span">
            <el-form-item label="客户名称:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.name" :disabled="jy"></el-input>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="企业名称:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.companyName" :disabled="jy"></el-input>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="用户名:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.username" :disabled="true"></el-input>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="创建时间:">
              <div :class="$style.code">
                <el-input :placeholder="formkhfull.createTime" :disabled="true"></el-input>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <el-divider content-position="left">客户类型</el-divider>
        <el-row>
          <el-col :span="12">
            <el-form-item label="代理商:">
              <div :class="$style.code">
                <el-select v-model="formkhfull.proxyType" :disabled="jy">
                  <el-option
                    v-for="item in lx"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id"
                  ></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12" v-if="formkhfull.proxyType!=4">
            <el-form-item label="区域:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.region" :disabled="jy"></el-input>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <el-divider content-position="left" v-if="formkhfull.proxyType!=5&formkhfull.proxyType!=6&formkhfull.proxyType!=1">上级客户</el-divider>
        <!-- 市级运营中心的下级 -->
        <el-row>
         <el-col :span="12" v-if="formkhfull.proxyType!=4&formkhfull.proxyType!=6&formkhfull.proxyType!=''&formkhfull.proxyType!=1&formkhfull.proxyType!=3&formkhfull.proxyType!=5">
            <el-form-item label="合资公司:">
              <div :class="$style.code">
                <el-select v-model="formkhfull.parentHZ" placeholder="请选择" clearable>
                  <el-option
                    v-for="item in hezigongsi"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  ></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
          </el-row>
        <!-- 市级一般代理的上级 -->
        <el-row>
          <el-col :span="12" v-if="formkhfull.proxyType!=4&formkhfull.proxyType!=6&formkhfull.proxyType!=''&formkhfull.proxyType!=1&formkhfull.proxyType!=2&formkhfull.proxyType!=5">
            <el-form-item label="市级运营中心:" >
              <div :class="$style.code">
                <el-select v-model="formkhfull.parentSJ" placeholder="请选择" :disabled="stop5" @change="getshiji" clearable>
                  <el-option v-for="item in shijiyunying" :key="item.id" :label="item.name" :value="item.name"></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12" v-if="formkhfull.proxyType!=4&formkhfull.proxyType!=6&formkhfull.proxyType!=''&formkhfull.proxyType!=1&formkhfull.proxyType!=2&formkhfull.proxyType!=5">
            <el-form-item label="合资公司:">
              <div :class="$style.code">
                <el-select v-model="formkhfull.parentHZ" placeholder="请选择" :disabled="stop4" @change="getFinds" clearable>
                  <el-option
                    v-for="item in hezigongsi"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  ></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <!-- 大客户的上级 -->
        <el-row>
          <el-col :span="12" v-if="formkhfull.proxyType!=6&formkhfull.proxyType!=1&formkhfull.proxyType!=''&formkhfull.proxyType!=2&formkhfull.proxyType!=3&formkhfull.proxyType!=5">
            <el-form-item label="市级一般代理:" >
              <div :class="$style.code">
                <el-select v-model="formkhfull.parentYB" placeholder="请选择" :disabled="stop3" @change="getyiban" clearable>
                  <el-option v-for="item in shijiyiban" :key="item.id" :label="item.name" :value="item.name"></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12" v-if="formkhfull.proxyType!=6&formkhfull.proxyType!=1&formkhfull.proxyType!=''&formkhfull.proxyType!=2&formkhfull.proxyType!=3&formkhfull.proxyType!=5">
            <el-form-item label="市级运营中心:" >
              <div :class="$style.code">
                <el-select v-model="formkhfull.parentSJ" placeholder="请选择" :disabled="stop2" @change="getshiji" clearable>
                  <el-option v-for="item in shijiyunying" :key="item.id" :label="item.name" :value="item.name"></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12" v-if="formkhfull.proxyType!=6&formkhfull.proxyType!=1&formkhfull.proxyType!=''&formkhfull.proxyType!=2&formkhfull.proxyType!=3&formkhfull.proxyType!=5">
            <el-form-item label="合资公司:">
              <div :class="$style.code">
                <el-select v-model="formkhfull.parentHZ" placeholder="请选择" :disabled="stop1" @change="getFinds" clearable>
                  <el-option
                    v-for="item in hezigongsi"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  ></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <el-divider content-position="left" v-if="formkhfull.stamp==='大客户'">数据共享</el-divider>
        <el-row v-if="formkhfull.stamp==='大客户'">
          <el-col :span="8">
            <span>是否共享：</span>
            <el-switch
              v-model="gongxiang"
              active-color="#13ce66"
              v-if="gongxiang===false"
              @change="queren"
              inactive-color="#ff4949"
            ></el-switch>
            <el-switch
              v-model="gongxiang"
              active-color="#13ce66"
              v-if="gongxiang===true"
              disabled
              inactive-color="#ff4949"
            ></el-switch>
          </el-col>
        </el-row>
        <el-divider content-position="left">联系方式</el-divider>
        <el-row>
          <el-col :span="12">
            <el-form-item label="联系人:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.contact"></el-input>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="联系电话:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.phone"></el-input>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="联系邮箱:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.email"></el-input>
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="联系地址:">
              <div :class="$style.code">
                <el-input v-model="formkhfull.address"></el-input>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
         <el-row>
          <el-col :span="12">
            <el-form-item label="专属客服:">
              <div :class="$style.code">
                <el-select v-model="formkhfull.service">
                  <el-option
                    v-for="item in optionsd"
                    :key="item.id"
                    :label="item.name"
                    :value="item.name"
                  ></el-option>
                </el-select>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        <el-divider content-position="left">区块链</el-divider>
        <el-form-item label="区块链账号:">
          <div :class="$style.code">
            <el-input v-model="formkhfull.account" :disabled="jy"></el-input>
          </div>
        </el-form-item>
        <el-divider content-position="left">备注</el-divider>
        <el-form-item label="备注:">
          <div :class="$style.code">
            <el-input v-model="formkhfull.remarks" type="textarea" autosize></el-input>
          </div>
        </el-form-item>
      </el-form>
      <span>
        <el-button @click="khfull = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisibles">确 定</el-button>
      </span>
    </el-dialog>
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
      name: '',
      optionsd:[],
      shijiyunying:[],
      shijiyiban:[],
      hezigongsi:[],
      options: [
        {
          id: 1,
          name: '省级运营中心'
        },
        {
          id: 2,
          name: '市级运营中心'
        },
        {
          id: 3,
          name: '市级一般代理商'
        },
        {
          id: 4,
          name: '大客户'
        },
        {
          id: 5,
          name: '清竹数据'
        }
      ],
      all: null,
      bigkh: null,
      shengjyyzx: null,
      sjyyzx: null,
      ybdl: null,
      value: '',
      ip: '',
      value1: '',
      tableData: [],
      currentRow: null,
      currentPage4: 1,
      total: null,
      full: false,
      labelPosition: 'right',
      formLabelAlign: {
      },
      khfull: false,
      formkhfull: {},
      gongxiang: false,
      leixing: '',
      stop1:false,
      stop2:false,
      stop3:false,
      stop4:false,
      stop5:false,
      accountName:'',
      // value:'',
      xiaji:[],
      lx: [
         {
          id: 1,
          name: '省级运营中心'
        },
        {
          id: 2,
          name: '市级运营中心'
        },
        {
          id: 3,
          name: '市级一般代理商'
        },
        {
          id: 4,
          name: '大客户'
        },
        {
          id: 5,
          name: '清竹数据'
        },
        {
          id:6,
          name:'合资公司'
        }
      ],
      jy: true,
      value2:'',
      value3:'',
      parentSJ:'',
      parentHZ:'',
      parentYB:'',
      sizes: 10,
      pages: 0,
      loading: true,
      name:''
    }
  },
  mounted () {
    this.getTotle()
    this.getCount()
    this.getFind()
    this.getshiji()
    this.getFinds()
    this.getyiban()
  },
  methods: {
    client(){
      this.$router.push({
        path:"client.vue"
      })
    },
    //市级运营中心
    getshiji(){
      if(this.formkhfull.parentSJ != null){
        this.stop1 = true
        this.stop3 = true
      }
      if(this.formkhfull.parentSJ == ""){
        this.stop4 = false
        this.stop5 = false
        this.stop1 = false
        this.stop2 = false
        this.stop3 = false
      }else if(this.formkhfull.parentSJ != null){
        this.stop4 = true
      }else{
        this.stop4 = false
        this.stop5 = false
      }
      this.$http.get(`modules/merchant/findCountry`,{
          params:{
          proxyType:2,
          // 'parentId':this.value2
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.shijiyunying = data
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
    //合资公司
    getFinds () {
      if(this.formkhfull.parentHZ != null){
        this.stop2 = true
        this.stop3 = true
      }
      if(this.formkhfull.parentHZ == ""){
        this.stop4 = false
        this.stop5 = false
        this.stop1 = false
        this.stop2 = false
        this.stop3 = false
      }else if(this.formkhfull.parentHZ != null){
        this.stop5 = true
      }else{
        this.stop4 = false
        this.stop5 = false
      }
      this.$http.get(`modules/merchant/findCountry`,{
          params:{
          proxyType:6,
          // 'parentId':this.value1
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.hezigongsi = data
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
    // 市级一般代理商
    getyiban(){
      if(this.formkhfull.parentYB != null){
        this.stop1 = true
        this.stop2 = true
      }else if(this.formkhfull.parentYB == ""){
        this.stop1 = false
        this.stop2 = false
        this.stop3 = false
        this.stop4 = false
        this.stop5 = false
      }

      //多余
      if(this.formkhfull.parentYB == ""){
        this.stop4 = false
        this.stop5 = false
        this.stop1 = false
        this.stop2 = false
        this.stop3 = false
      }else if(this.formkhfull.parentYB != null){
        // this.stop5 = true
      }else{
        // this.stop4 = false
        // this.stop5 = false
      }
      this.$http.get(`modules/merchant/findCountry`,{
          params:{
          proxyType:3,
          // 'parentId':this.value3
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.shijiyiban = data
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
    getTotle () {
      this.$http.get(`modules/merchant/list`, {
        params: {
          size: this.sizes,
          page: this.pages
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.loading = false
          this.tableData = []
          this.tableData = data.content
          this.total = data.total
          this.tableData.forEach(item => {
            if (item.province === null) {
              item.region = ''
            } else if (item.city === null) {
              item.region = item.province
            } else {
              item.region = item.province + "\\" + item.city
            }
          })
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
    getFind () {
      this.$http.get(`sys/user/findService`).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.optionsd = data
          this.getTotle()
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
    getCount () {
      this.$http.get(`pc/merchant/merchantCount`).then(res => {
        // console.log(res)
        var { code, data } = res.data
        if (code === 1000) {
          this.all = data.市级运营中心 + data.省级运营中心 + data.市级一般代理商 + data.大客户
          this.bigkh = data.大客户
          this.shengjyyzx = data.省级运营中心
          this.sjyyzx = data.市级运营中心
          this.ybdl = data.市级一般代理商 
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息')
      })
    },
    search () {
      console.log(this.value)
      this.pages = 0
      this.currentPage4 = 1
      this.$http.get(`modules/merchant/list`, {
        params: {
          size: this.sizes,
          page: this.pages,
          search: this.name,
          proxyType: this.value,
        }
      }).then(res => {
        console.log('ss',this.pages)
        var { code, data } = res.data
        if (code === 1000) {
          this.tableData = data.content
          this.total = data.total
          this.tableData.forEach(item => {
            if (item.province === null) {
              item.region = ''
            } else if (item.city === null) {
              item.region = item.province
            } else {
              item.region = item.province + "\\" + item.city
            }
          })
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
      this.$http.get(`modules/merchant/list`, {
        params: {
          size: val,
          page: this.pages,
          proxyType: this.value,
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.tableData = data.content
          this.total = data.total
          this.tableData.forEach(item => {
            if (item.province === null) {
              item.region = ''
            } else if (item.city === null) {
              item.region = item.province
            } else {
              item.region = item.province + "\\" + item.city
            }
          })
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
      this.pages = val - 1
      this.$http.get(`modules/merchant/list`, {
        params: {
          size: this.sizes,
          page: val - 1,
          proxyType: this.value,
        }
      }).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.tableData = []
          this.tableData = data.content
          this.total = data.total
          this.tableData.forEach(item => {
            if (item.province === null) {
              item.region = ''
            } else if (item.city === null) {
              item.region = item.province
            } else {
              item.region = item.province + "\\" + item.city
            }
          })
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
    dialogVisibles(){
      console.log('xiajsssi',this.xiaji.name)
      console.log('客服',this.formkhfull.service)
      this.khfull = false
        let info = {
          'contact':this.formkhfull.contact,//联系人
          'phone':this.formkhfull.phone,//联系电话
          'email':this.formkhfull.email,//联系邮箱
          'address':this.formkhfull.address,//联系地址
          'id':this.formkhfull.id,//id
          'service':this.formkhfull.service,//客服
          'parentName':this.formkhfull.parentYB||this.formkhfull.parentSJ||this.formkhfull.parentHZ,
        }
      this.$http.post(`modules/merchant/update`,info).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.khfull = false
          this.getTotle()
            this.$message({
              message: '修改成功',
              type: 'success'
            });
        } else {
          this.$message.error(res.data.message);
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    handleEdit (index, row) {
      console.log(index, row);
      this.formLabelAlign = row
      this.full = true
    },
    Enable (index, row) {
      this.$http.put(`modules/merchant/update/${row.id}/status/1`).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.getTotle()
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    Disable (index, row) {
      this.$http.put(`modules/merchant/update/${row.id}/status/2`).then(res => {
        var { code, data } = res.data
        if (code === 1000) {
          this.getTotle()
        }
      }).catch((err) => {
        console.log('错误信息' + err)
      })
    },
    handleDelete (index, row) {
        this.$confirm('确认要重置密码?, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let info = {
            'id': row.id
          }
          this.$http.post('modules/merchant/resetPassword', info).then(res => {
            var { code, data } = res.data
            if (code === 1000) {
              this.$message({
                message: '已重制密码',
                type: 'success'
              });
            } else {
              this.$message.error(res.data.message);
            }
          }).catch(function (error) {
            console.log('错误信息' + error)
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });
        });
      },
    handleClose () {
      this.full = false
    },
    khname (index, row) {
      console.log(row)
      this.$http.get(`modules/merchant/${row.id}`,{params:{
      }}).then(res => {
        console.log(res)
        var { code, data } = res.data
        if (code === 1000) {
          console.log(data)
          if (row.proxy_type === 4) {
            console.log(1)
            this.jy = true
            this.khfull = true
            this.formkhfull = data
            // this.xiaji = data.findProxyTypeList
            // console.log('xiaji',this.xiaji.name)
          } else {
            console.log(2)
            this.jy = true
            this.khfull = true
            this.formkhfull = data
            // this.xiaji = data.findProxyTypeList
            // let arr = []
            // for(let i = 0; i<arr.length;i++){

            // }
            console.log('xiajia',this.xiaji)
             if (this.formkhfull.province === null) {
              this.formkhfull.region = ''
            } else if (this.formkhfull.city === null) {
              this.formkhfull.region = data.province
            } else {
              this.formkhfull.region = data.province + "\\" + data.city
            }
          }
        } else if (code == 2001) {
          this.$message.error(res.data.message);
          window.sessionStorage.clear();
          window.localStorage.clear();
          this.$router.push('/')
        } else {
          this.$message.error(res.data.message);
        }
      }).catch(function (error) {
        console.log('错误信息' + error)
      })

      // this.khfull = true
      // this.formkhfull = row
      // this.$router.push('/index/clientele.vue/clientelxq.vue')
    },
    //赋权
    empower(index,row){
      console.log(row)
      this.$confirm('确认要赋权吗?, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http.get('modules/merchant/updateauth', {
              params:{
                'accountName':row.accountName
                // accountName:'qzcrrohrzez3'
            }
          }).then(res => {
            var { code, data } = res.data
            if (code === 1000) {
              this.$message({
                message: '已赋权',
                type: 'success'
              });
            } else {
              this.$message.error(res.data.message);
            }
          }).catch(function (error) {
            console.log('错误信息' + error)
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });
        });
    },
    queren () {
      this.$confirm('此操作将永久生效, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.gongxiang = true
      }).catch(() => {
        this.gongxiang = false
      });
    },
    handleClose1 () {
      console.log(1)
      this.khfull = false
    }
  }
}
</script>

<style lang='scss' module>
.name {
  width: calc(100% - 80%);
  float: left;
}
.office {
  width: calc(100% - 80%);
  float: left;
  margin-left: 20px;
}
.ip {
  width: calc(100% - 80%);
  float: left;
  margin-left: 20px;
}
.date {
  width: calc(100% - 80%);
  float: left;
  margin-left: 20px;
}
.table {
  margin-top: 20px;
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
.f_khnam {
  cursor: pointer;
  color: #252cdc;
}
.f_khxq {
  .el-dialog {
    margin-top: 0;
  }
}
p {
  margin: 0;
  font-family: "MicrosoftYaHei", "微软雅黑";
  font-size: 14px;
  color: #999999;
}
.code {
  float: left;
}
.f_gx {
  float: left;
  margin-top: 10px;
}
</style>
