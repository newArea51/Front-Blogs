<template>
  <section class="flex">
    <div class="info flex-1 left-content">
      <div class="title flex flex-align-center">
        <icon name="user-secret" scale="1.2" style="color:#00AACD"></icon>
        <span class="title-text">关于我</span>
        <span class="title-label">About me</span>
      </div>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm bgbox box">
        <el-form-item label="站长头像" prop="imageUrl" v-loading.body="loadingAvatarUpload" required>
          <el-upload
            class="avatars-uploader"
            action="http://api.55lover.com/api/upload"
            :show-file-list="false"
            :disabled="true"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="ruleForm.imageUrl" :src="ruleForm.imageUrl" class="avatars">
            <i v-else class="el-icon-plus avatars-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="公司名称" prop="company">
          <el-input v-model="ruleForm.company" :disabled="true"></el-input>
        </el-form-item>
        </el-form-item>
        <el-form-item label="出生日期" required>
          <el-col :span="14">
            <el-form-item prop="borndate">
              <el-date-picker type="date" placeholder="选择日期" v-model="ruleForm.borndate" :disabled="true" style="width: 100%;"></el-date-picker>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="性别" prop="gender" required>
          <el-radio-group v-model="ruleForm.gender" :disabled="true">
            <el-radio label="男"></el-radio>
            <el-radio label="女"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="个人介绍" prop="desc">
          <el-input type="textarea" v-model="ruleForm.desc" :disabled="true" :autosize="{ minRows: 3, maxRows: 6}"></el-input>
        </el-form-item>
        <el-form-item label="个人网站" prop="site">
          <el-input v-model="ruleForm.site" :disabled="true"></el-input>
        </el-form-item>
        <!-- <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">立即修改</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
        </el-form-item> -->
      </el-form>
    </div>
    <div class="right-content">
      <asides></asides>
    </div>
  </section>
</template>
<script>
  import Asides from '@/components/Aside'
  import 'vue-awesome/icons'
  import Icon from 'vue-awesome/components/Icon'
  export default {
    components: {
      Asides,
      Icon
    },
    data() {
      return {
        loadingAvatarUpload: false,
        ruleForm: {
          company: '',
          borndate: '',
          type: [],
          desc: '',
          // 本地
          imageUrl: '',
          // 服务器avatar地址
          avatar: '',
          site: '',
          gender: ''
        },
        rules: {
          imageUrl:[
            { required: true, message: '请上传头像' }
          ],
          company: [
            { required: true, message: '请输入公司名称', trigger: 'blur' },
            { min: 2, max: 50, message: '长度在 2 到 50 个字符', trigger: 'blur' }
          ],
          borndate: [
            { type: 'date', required: true, message: '请选择出生日期', trigger: 'change' }
          ],
          desc: [
            { required: true, message: '请填写个人介绍', trigger: 'blur' }
          ],
          gender :[
            {required: true, message: '请选择性别'}
          ]
        }
      };
    },
    methods: {
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.updateInfo()
          } else {
            return false;
          }
        });
      },
      resetForm(formName) {
        // this.ruleForm.avatar = ''
        this.$refs[formName].resetFields();
      },
      handleAvatarSuccess(res, file) {
        this.loadingAvatarUpload= false
        this.ruleForm.imageUrl = URL.createObjectURL(file.raw);
        this.ruleForm.avatar = res.imgUrl
      },
      beforeAvatarUpload(file) {
        this.loadingAvatarUpload= true
        const isJPG = file.type === 'image/jpeg';
        const isLt2M = file.size / 1024 / 1024 < 2;

        if (!isJPG) {
          this.$message.error('上传头像图片只能是 JPG 格式!');
          this.loadingAvatarUpload= false
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
          this.loadingAvatarUpload= false
        }
        return isJPG && isLt2M;
      },
      updateInfo () {
        if (this.ruleForm.gender === '男') {
          var gender = 1
        } else {
          var gender = 0
        }
        var para = {
          id: 'd17692be-eca7-41ef-87df-aef4313e2b02',
          avatar: this.ruleForm.avatar,
          desc: this.ruleForm.desc,
          gender: gender,
          company: this.ruleForm.company,
          site: this.ruleForm.site,
          borndate: Coms.getCommonTime(this.ruleForm.borndate)
        }
        axios.post('/api/updateInfo', Qs.stringify(para))
        .then(res => {
          if (res.data.isSuc) {
            this.$message.success(res.data.message)
            let r = JSON.stringify(res.data.result)
            window.localStorage.setItem('55lover_reader', r)
            this.$router.go(-1)
          } else {
            this.$message.error(res.data.message)
          }
        })
        .catch(err => {})
      }
    },
    mounted () {
      $('title').html('站长信息_关于我_邓鹏博客')
      axios.get('/api/getInfo/d17692be-eca7-41ef-87df-aef4313e2b02')
      .then(res => {
        // console.log(res)
        if (res.data.isSuc && res.data.result) {
          let r = res.data.result
          this.ruleForm.imageUrl = r.avatar
          this.ruleForm.avatar = r.avatar
          this.ruleForm.desc = r.desc
          this.ruleForm.borndate = new Date(r.borndate)
          this.ruleForm.site = r.site || ''
          this.ruleForm.company = r.company
          if (r.gender == 1) {
            this.ruleForm.gender = '男'
          } else {
            this.ruleForm.gender = '女'
          }
        }
      })
      .catch(err => {})
    }
  }
</script>

<style>
  .demo-ruleForm .el-form-item__label, 
  .demo-ruleForm .el-input.is-disabled .el-input__inner, 
  .demo-ruleForm .el-textarea.is-disabled .el-textarea__inner,
  .demo-ruleForm .el-radio__input.is-disabled+span.el-radio__label{
    color: #444
  }
  .avatars-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatars-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatars-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatars {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>

<style scoped>
  .info {
    /*width: 400px;*/
    margin: 0 auto 40px;
  }
  .logo  {
    margin-bottom: 40px;
    text-align: center;
  }
</style>