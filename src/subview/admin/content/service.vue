<template>
  <div class="page">
    <div class="search_container">
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item v-for="(item, index) in searchList"  :key="index"> 
          <el-input v-model="item.keyword" :placeholder="item.label"></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model="auditStatus" placeholder="请选择状态">
            <el-option label="不限" value=""></el-option>
            <el-option label="已审核" value="1"></el-option>
            <el-option label="未审核 " value="0"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item><!-- label="分类" -->
            <el-cascader
              placeholder="内容分类"
              expand-trigger="hover"
              :options="catagory"
              filterable
              change-on-select
              @change="handlecatagotyChange"
            ></el-cascader>
        </el-form-item>
        <el-form-item>
          <el-date-picker
            v-model="date"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            format="yyyy 年 MM 月 dd 日"
            value-format="yyyy-MM-dd">
          </el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">搜索</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="table_container">
      <el-table
        :data="tableData"
        style="width: 100%" @selection-change="handleSelectionChange">
        <el-table-column
          type="selection">
        </el-table-column>
        <el-table-column
          label="ID" sortable>
          <template slot-scope="scope">
            <span>{{scope.row.shopServiceId}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="名称">
          <template slot-scope="scope">
            <span>{{scope.row.title}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="用户UID">
          <template slot-scope="scope">
            <span  v-if="scope.row.memberShop.member">{{scope.row.memberShop.member.uid}}</span>
            <span v-else></span>
          </template>
        </el-table-column>
        <el-table-column     
          label="用户名">
          <template slot-scope="scope">
            <span  v-if="scope.row.memberShop.member">{{scope.row.memberShop.member.userName}}</span>
            <span v-else></span>
          </template>
        </el-table-column>
        <el-table-column
          label="分类">
          <template slot-scope="scope">
            <span  v-if="scope.row.categoryFirst">{{scope.row.categoryFirst.categName}}</span>
            <span v-else></span>
            <span  v-if="scope.row.categorySecond">>>{{scope.row.categorySecond.categName}}</span>
            <span v-else></span>
            <span  v-if="scope.row.categoryThree">>>{{scope.row.categoryThree.categName}}</span>
            <span v-else></span>
          </template>
        </el-table-column>
        <!-- <el-table-column
          label="出售金额">
          <template slot-scope="scope">
            <span>{{scope.row.realName}}</span>
          </template>
        </el-table-column> -->
        <el-table-column
          label="状态 ">
          <template slot-scope="scope">
            <span :class="{statusGreen:true , statusGray:false }" v-if="scope.row.status=='1'">{{scope.row.status | serviceStatus}}</span>
            <span :class="{statusGreen:false , statusGray:true }" v-else>{{scope.row.status | serviceStatus}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="推荐 ">
          <template slot-scope="scope">
            <span :class="{topGreen: true }" v-if="scope.row.isTop=='1'">{{scope.row.isTop | serviceTop}}</span>
            <span :class="{topGreen: false}" v-else>{{scope.row.isTop | serviceTop}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="发布时间">
          <template slot-scope="scope">
            <span>{{scope.row.releaseTime}}</span>
          </template>
        </el-table-column>
          <el-table-column label="操作" :min-width="150">
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
        <el-row>
          <el-pagination
            layout="prev, pager, next"
            :total="paginations.total"
            :page-size="paginations.page_size"
            :current-page.sync="paginations.page_current"
            @current-change="handleCurrentChange">
        </el-pagination>
      </el-row>
      <el-row type="flex" class="row-bg" justify="center">
        <el-button @click="batchAudit(1)">审核</el-button>
        <el-button @click="batchAudit(0)">取消审核</el-button>
        <el-button @click="RecommendOrNot(1)">推荐</el-button>
        <el-button @click="RecommendOrNot(0)">取消推荐</el-button>
        <el-button type="danger" @click="batchDelete">删除</el-button>
    </el-row>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import { mapState } from 'vuex'
export default {
  data() {
    return {
        searchList:[   //普通搜索框
          {
            label: "服务ID",
            keyword:""
          },
          {
            label: "用户UID",
            keyword:""
          },
          {
            label: "标题",
            keyword:""
          }
        ],
     catagory: [],
     catagoryKey:[],
     auditStatus:'',   //审核状态
     date:"", //时期范围
     tableData: [],
     multipleSelection:[],
      paginations: {
          page_current: 1,  // 当前位于哪页
          page_size: 9,   // 1页显示多少条
          // total: 30,        // 总数
      },
      statusGreen:true,
      statusGray:true,
      topGreen:true,
    }
  },
  created() {
      this.GLOBAL.getCategory((data)=>{
              // 先将json对象转为json字符串，再替换你要替换的属性名，最后再转为json对象
              // console.log("getCategory", data.categoryFirsts)
              data = JSON.parse(JSON.stringify(data).replace(/categFirstId|categSecondId|categThreeId/g,"value"));
              data = JSON.parse(JSON.stringify(data).replace(/categName/g,"label"));
              data = JSON.parse(JSON.stringify(data).replace(/categorySeconds|categoryThrees/g,"children"));
              this.catagory = data.categoryFirsts
    })
    this.handleCurrentChange()
  },
  filters :{
    serviceTop(value){
      if(value == '1'){
        return "推荐"
      }else{
        return "未推荐"
      }
    },
    serviceStatus(value){
      if(value == '1'){
        return "已审核"
      }else{
        return "未审核"
      }
    }
  },
  methods: {
    changeColor() {
      if(this.status == '3'){
          this.statusGreen = true
      }else{
          this.statusGreen = true
      }
    },
    handlecatagotyChange(val){
      this.catagoryKey=val
    },
    onSubmit() {
      // console.log(this.date)
        if( this.date==""&&this.catagoryKey=="" &&this.auditStatus==""&&this.searchList[0].keyword=="" && this.searchList[1].keyword==""&& this.searchList[2].keyword==""){
          this.$message.error("搜索框不能为空")
          this.handleCurrentChange()
          return 0
        }
        this.$http.post(this.$api.serviceSearch,
            this.Qs.stringify({
              "shopServices.serviceId": this.searchList[0].keyword,
              "shopServices.uid": this.searchList[1].keyword,
              "shopServices.title": this.searchList[2].keyword,
              "shopServices.status": this.auditStatus,
              "shopServices.startTime": this.date[0],
              "shopServices.endTime":  this.date[1],
              "shopServices.categFirstId": this.catagoryKey[0],
              "shopServices.categSecondId":  this.catagoryKey[1],
              "shopServices.categThreeId":  this.catagoryKey[2],
              "page.pageNo":this.paginations.page_current,
              "page.pageSize":this.paginations.page_size
            }))
          .then((response) => {
            if (response.data.errorMessage == "您还未登录，请先登录") {
              this.$router.push('/login');
            } else {
              // console.log(response)
              this.tableData = response.data.shopServices
            }
          })
          .catch(function(error) {
            console.log(error)
          })
      },
      handleEdit(index, row) {
        console.log(index, row);
        this.$router.push('/main/service/'+ row.shopServiceId)
      },
      handleDelete(index, row) {
        this.$http.post(this.$api.servicebatchDelete,
            this.Qs.stringify({
              "ids": row.shopServiceId
            }))
          .then((response) => {
            if (response.data.errorMessage == "您还未登录，请先登录") {
              this.$router.push('/login')
            }else if (response.data.errorMessage) {
              this.$message.error(response.data.errorMessage)
            }else if (response.data.result==1) {
              this.tableData.splice(index, 1);
              if(response.data.result==1){
                this.$message({
                   message: '成功删除服务:  ' + row.userName,
                    type: 'success' 
                  })
               this.handleCurrentChange()
              }
            }
          })
          .catch(function(error) {
            console.log(error)
          })
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;
        // console.log(val)
      },
    RecommendOrNot(opera) {
      let ids = []
      this.multipleSelection.forEach(function(value, index, array) {
        ids.push(array[index].shopServiceId)
      })
      console.log("ids",ids)
      this.$http.post(this.$api.serviceTop,
          this.Qs.stringify({
            "ids": ids,
            "isRecommend": opera
          }))
        .then((response) => {
          if (response.data.errorMessage == "您还未登录，请先登录") {
            this.$router.push('/login');
          } else if (response.data.result==1){
            this.$message({
                message: '推荐成功! ',
                type: 'success' 
              })
            this.handleCurrentChange()
          }
        })
        .catch(function(error) {
          console.log(error)
        })
      },
      batchDelete() {
        let ids = []
        this.multipleSelection.forEach(function(value, index, array) {
          ids.push(array[index].shopServiceId)
        })
        this.$http.post(this.$api.servicebatchDelete,
            this.Qs.stringify({
              "ids": ids
            }))
          .then((response) => {
            if (response.data.errorMessage == "您还未登录，请先登录") {
              this.$router.push('/login')
            } else   if (response.data.errorMessage) {
                this.$message.error(response.data.errorMessage)
            } else if(response.data.result==1) {
              this.$message({
                  message: '成功删除服务!  ',
                  type: 'success' 
                })
              this.handleCurrentChange()
            }
          })
          .catch(function(error) {
            console.log(error)
          })
      },
      batchAudit(status) {
        let ids = []
        this.multipleSelection.forEach(function(value, index, array) {
          ids.push(array[index].shopServiceId)
          console.log("ids",ids)
        })
        this.$http.post(this.$api.servicebatchAudit,
            this.Qs.stringify({
              "ids": ids,
              "isAuditPass": status
            }))
          .then((response) => {
            if (response.data.errorMessage == "您还未登录，请先登录") {
              this.$router.push('/login')
            } else   if (response.data.errorMessage) {
                this.$message.error(response.data.errorMessage)
          } else if (response.data.result==1){
              this.$message({
                  message: '成功审核!  ',
                  type: 'success' 
                })
                this.handleCurrentChange()
            }
          })
          .catch(function(error) {
            console.log(error)
          })
      },
      handleCurrentChange(page) {
        console.log(this.paginations.page_current)
        this.$http.post(this.$api.servicefindAll,
        this.Qs.stringify({
          "page.pageNo":this.paginations.page_current,
          "page.pageSize":this.paginations.page_size
        }))
       .then((response) => { 
          if(response.data.errorMessage == "您还未登录，请先登录"){
              this.$router.push('/login');
          }else{
            console.log(response)
              this.tableData = response.data.shopServices
          }
        })
        .catch(function (error) {
          console.log(error)
        })
      }
  },
  components: {

  }
}
</script>

<style scoped lang="stylus">
a
    color: #000
a:-webkit-any-link
    text-decoration none
.statusGreen
    color: white
    background: #159715b0
    padding: 5px
.statusGray
    color: white
    background: #c0c4ccb5
    padding: 5px
.topGreen
    color: white
    background: #159715b0
    padding: 5px
</style>
