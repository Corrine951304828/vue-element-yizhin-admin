  <template>
  <div class="page">
    <el-form ref="form" :rules="rules"  :model="form" label-width="120px">
      <el-form-item label="省份名称" prop="cityName">
        <el-input v-model="form.cityName"></el-input>
      </el-form-item>        
      <el-form-item>
          <el-button type="primary" @click="submitForm('updateGroup')">修改</el-button>
          <el-button @click="resetForm('updateGroup')">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data() {
    return {
        form: {
          cityName: ''
        },
        rules: {
          cityName: [
            { required: true, message: '请输入市级名', trigger: 'blur' }
          ]
        },
    }
  },
  created() {
    this.initData()
  },
  methods: {
      initData(page) {
        this.$http.post(this.$api.City_search,this.Qs.stringify({
               "cityCategoryJ.cityCategoryId":this.$route.params.cataid,
              "page.pageNo":1,
              "page.pageSize":100
        }))
       .then((response) => { 
          if(response.data.errorMessage == "您还未登录，请先登录"){
              this.$router.push('/login');
          } else if (response.data.errorMessage){
              this.$message.error(response.data.errorMessage)
          }else{
              this.form = response.data.cityCategorys[0]
          }
        })
        .catch(function (error) {
          console.log(error)
        })
      },
      submitForm(formName) {
            this.$http.post(this.$api.City_update,
            this.Qs.stringify({
              "cityCategoryJ.cityCategoryId": this.$route.params.cataid,
              "cityCategoryJ.cityName": this.form.cityName
            }))
            .then((response) => {
              if(response.data.result==1){
                this.$message({
                   message: '成功修改市级!',
                    type: 'success' 
                  })
              }else if(response.data.errorMessage == "您还未登录，请先登录"){
                 this.$router.push('/login');
              }else{
                this.$message.error(response.data.errorMessage)
              }
            }).catch((err)=>{
                this.$message.error('请求失败!')
            })
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      }
  }
}
</script>

<style scoped lang="stylus">
.el-input
    width: 40%
</style>

