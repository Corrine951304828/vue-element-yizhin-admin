<template>
  <div class="page">
    <div class="search_container">
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item v-for="(item, index) in searchList"  :key="index"> 
          <el-input v-model="item.keyword" :placeholder="item.label"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="roleSearch">搜索</el-button>
        </el-form-item>
        <el-form-item style="float: right;">
          <router-link to="/main/role/add"><el-button type="primary" icon="el-icon-plus"></el-button></router-link>
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
            <span>{{scope.row.authGroupId}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="名称">
          <template slot-scope="scope">
            <span>{{scope.row.groupName}}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="状态">
          <template slot-scope="scope">
            <span>{{scope.row.status | roleStatus}}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.$index, scope.row)" v-show="(scope.row.authGroupId  !=1)&&(scope.row.authGroupId  !=2)&&(scope.row.authGroupId  !=3)">删除</el-button>
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
        <el-button type="danger" @click="batchDelete">删除</el-button>
    </el-row>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data() {
    return {
        searchList:[   //普通搜索框
          {
            label: "用户组id",
            keyword:""
          },
          {
            label: "用户组名",
            keyword:""
          }
        ],
      tableData: [],
      multipleSelection: [],   //多选数据
      paginations: {
          page_current: 1,  // 当前位于哪页
          page_size: 9,   // 1页显示多少条
          // total: 30,        // 总数
      }
    }
  },
  created() {
    this.handleCurrentChange()
  },
  filters :{
    roleStatus: function(value){
    if(value == '0'){
      return " "
      }else{
        return "开启"
      }
    }
  },
  methods: {
    roleSearch() {
        if(this.searchList[0].keyword=="" && this.searchList[1].keyword==""){
          this.$message.error("搜索框不能为空")
          this.handleCurrentChange()
          return 0
        }
        this.$http.post(this.$api.roleSearch,
            this.Qs.stringify({
              "role.authGroupId": this.searchList[0].keyword,
              "role.groupName": this.searchList[1].keyword,
              "page.pageNo":this.paginations.page_current,
              "page.pageSize":this.paginations.page_size
            }))
          .then((response) => {
            if (response.data.errorMessage == "您还未登录，请先登录") {
              this.$router.push('/login');
            } else if (response.data.errorMessage){
                this.$message.error(response.data.errorMessage)
            } else {
              console.log(response)
              this.tableData = response.data.authGroups
            }
          })
          .catch(function(error) {
            console.log(error)
          })
      },
      handleEdit(index, row) {
        console.log(index, row);
        this.$router.push('/main/role/edit/'+ row.authGroupId)
      },
      handleDelete(index, row) {
         this.$http.post(this.$api.roleDelete,this.Qs.stringify({
            "role.authGroupId":row.authGroupId
          }))
        .then((response) => { 
          if(response.data.result == 1){
              this.tableData.splice(index, 1);
                this.$message({
                   message: '成功删除角色:  ' + row.groupName,
                    type: 'success' 
                  })
            }else if(response.data.errorMessage == "您还未登录，请先登录"){
                this.$router.push('/login')
            } else{
                this.$message.error(response.data.errorMessage)
            }
          })
          .catch(function (error) {
            console.log(error)
          })
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;
        // console.log(val)
      },
      batchDelete() {
        let ids = []
        this.multipleSelection.forEach(function(value, index, array) {
          ids.push(array[index].authGroupId)
          console.log("ids",ids)
        })
        this.$http.post(this.$api.rolebatchDelete,
            this.Qs.stringify({
              "ids": ids
            }))
          .then((response) => {
            if (response.data.errorMessage == "您还未登录，请先登录") {
              this.$router.push('/login')
            } else   if (response.data.errorMessage) {
                this.$message.error(response.data.errorMessage)
            }else{
              this.$message({
                  message: '成功删除角色!  ',
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
        this.$http.post(this.$api.rolefindAll,
        this.Qs.stringify({
          "page.pageNo":this.paginations.page_current,
          "page.pageSize":this.paginations.page_size
        }))
       .then((response) => { 
          if(response.data.errorMessage == "您还未登录，请先登录"){
              this.$router.push('/login');
          } else if (response.data.errorMessage){
              this.$message.error(response.data.errorMessage)
          }else{
            console.log(response)
            this.tableData = response.data.authGroups
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
</style>
