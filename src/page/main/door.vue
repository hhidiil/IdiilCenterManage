<template>
  <el-container>
    <el-header class="door">
      <el-row>
        <el-col :span="4"><img src="../../../static/images/logo.png"></el-col>
        <el-col :span="20"><div class="loginButton"></div></el-col>
      </el-row>
    </el-header>
    <el-main>
      <img src="../../../static/images/idiilcenterbackground.jpg" class="HomeBanner" />
      <section class="form_contianer">
        <div class="senctionblock">
          <el-row>
            <el-col :span="24"><div class="loginButton">登 录</div></el-col>
          </el-row>
          <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="80px" class="demo-ruleForm">
            <el-form-item label="用户名" prop="name">
              <el-input type="text" v-model="ruleForm.name"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="pass">
              <el-input type="password" v-model="ruleForm.pass"></el-input>
            </el-form-item>
            <el-form-item label="角色" prop="role">
              <el-radio-group v-model="ruleForm.role">
                <el-radio label="1">学生</el-radio>
                <el-radio label="2">教师</el-radio>
                <el-radio label="3">管理员</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-button @click="loginEnter('ruleForm')" class="enterButton">登 录</el-button>
          </el-form>
        </div>
      </section>
    </el-main>
  </el-container>
</template>

<script type="text/ecmascript-6">
  import {doLogin,doTestLogin,registerUser} from '../../api/user'
  import classData from '../../data/classlist'
  import {setStore,getStore,clearStore,setSession,getSession} from '../../config/publicMethod'
  import {filterWebUrl,filterWebUrl2} from '../../config/methods'
  export default {
    name: 'door',
    data () {
      var validatePass = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请输入密码'));
        } else {
          if (this.ruleForm.checkPass !== '') {
            this.$refs.ruleForm.validateField('checkPass');
          }
          callback();
        }
      };
      var validatePass2 = (rule, value, callback) => {
          if (value === '') {
            callback(new Error('请再次输入密码'));
          } else if (value !== this.ruleForm.pass) {
            callback(new Error('两次输入密码不一致!'));
          } else {
            callback();
          }
      }
      return {
        ClassUserList:null,
        show2: true,
        btnChangeName:'登录',
        inputName: '',
        inputPassword:'',
        allUserList:[],
        ruleForm: {
          name:'',
          phone:'',
          pass:'',
          checkPass:'',
          role:null,
        },
        ruleForm1: {
          name:'',
          phone:'',
          pass:'',
          checkPass:'',
          role:null,
        },
        rules: {
          name: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
          ],
          phone: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { min: 11, max: 11, message: '请输入11位手机号', trigger: 'blur' }
          ],
          role:[
            { required: true, message: '请选择角色身份', trigger: 'change' }
          ],
          pass: [
            {required: true,  validator: validatePass, trigger: 'blur' }
          ],
          checkPass: [
            {required: true,  validator: validatePass2, trigger: 'blur' }
          ]
        }
      }
    },
    methods:{
      async loginEnter(formName){

        //查询用户信息
        const dataUserList = await doTestLogin('/static/ClassUserList.json');
        setStore("ClassUserList",dataUserList);
        this.ClassUserList = dataUserList;

        this.$refs[formName].validate(async(valid) => {
          if (valid) {
            let userinfo = null;
            let fromflag = this.fromFlag;//true为真实数据，false为本地数据
            console.log('55555555555555555555555555555555555555555555555',fromflag)
            if(fromflag){
              const result = await doLogin(this.ruleForm)//查询用户信息
              console.log("注册结果------->",result)
              if(result.code == 200){
                if((result.data[0])){
                  userinfo = {
                    userId:result.data[0].userId,
                    userName: result.data[0].userName,
                    centerId:result.data[0].centerId
                  };
                  if(this.ruleForm.role == '1'){//学生登录
                    setSession("accessToken",true);//设置登录状态的值
                    setStore("userInfo",JSON.stringify(userinfo))
                    this.$router.replace({ name: 'homeStudent', params: userinfo})
                  }else if(this.ruleForm.role == '2'){//教师登录
                    if(getStore("userInfo")){//已经有值
                      if(JSON.parse(getStore("userInfo")).userId != userinfo.userId){//和上次的登录人不一样
                        clearStore();
                        //由于清除了所有的 所以现在在存一遍需要的
                        const dataUserList = await doTestLogin('/static/ClassUserList.json');
                        setStore("ClassUserList",dataUserList);
                      }
                    }
                    setSession("accessToken",true);//设置登录状态的值
                    setStore("userInfo",JSON.stringify(userinfo))
                    this.$router.replace({ name: 'homeTeacher', params: userinfo})
                  }else {
                    setSession("accessToken",true);//设置登录状态的值
                    console.log("555555555555555555555555555555555555555555555555555",result.data[0])
                    setStore("manageUser",result.data[0])
                    let origin = window.location.origin;
                    window.location.href = origin + "/manage.html#/home";//跳转到数学派课系统
                  }
                }else {
                  alert("用户名或密码错误")
                }
              }else {
                console.error('登录失败!!');
              }
            }else {//本地数据
              let flag = false;
              if(this.ruleForm.role == '1'){//学生
                this.ClassUserList.StudentList.forEach((item)=>{
                  if(item.userName == this.ruleForm.name){
                    if(item.Password == this.ruleForm.pass){
                      flag = true;
                      userinfo = item;
                    }
                  }
                })
              }
              if(this.ruleForm.role == '2'){//教师
                this.ClassUserList.InstructorList.forEach((item)=>{
                  if(item.userName == this.ruleForm.name){
                    if(item.Password == this.ruleForm.pass){
                      flag = true;
                      userinfo = item;
                    }
                  }
                })
              }
              if(flag){
                setSession("accessToken",true);//设置登录状态的值
                setStore("userInfo",JSON.stringify(userinfo))
                if(this.ruleForm.role == '1'){
                  this.$router.replace({ name: 'homeStudent', params: userinfo})
                }else if(this.ruleForm.role == '2'){//教师登录
                  this.$router.replace({ name: 'homeTeacher', params: userinfo})
                }else {
                  alert("跳转到管理员页面")
                }
              }else {
                alert("用户名或密码错误")
              }
            }
          } else {
            console.error('error login submit!!');
            return false;
          }
        });
      },
      registerSubmitForm(formName){
        this.$refs[formName].validate(async(valid) => {
          if (valid) {
            console.log("角色------->",this.ruleForm)
            const data = await registerUser(this.ruleForm)
            console.log("注册结果------->",data)
          } else {
            console.error('error submit!!');
            return false;
          }
        });
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less" type="text/less">
  @import '../../assets/mixin.less';
  .door{
    background-color: #5c4241;
  }
  .form_contianer .el-row{
    margin: 15px 0;
  }
  .el-main{
    padding: 0;
  }
  .manage_tip{
    position: absolute;
    width: 100%;
    left:0;
    p{
      font-size: 34px;
    }
    button{
      width: 200px;
      margin-top: 20px;
      font-size: 18px;
    }
  }
  .senctionblock{
    position: relative;
    width: 100%;
    background-color: white;
  }
  .HomeBanner{
    width: 100%;
    height: -webkit-fill-available;
    background-size: cover;
  }
  .form_contianer{
    width: 320px;
    min-height: 210px;
    position: absolute;
    top: 25%;
    left: 75%;
    margin-top: -105px;
    margin-left: -160px;
    padding: 25px;
    border-radius: 5px;
    text-align: center;
    background-color: #fff;
    .clickchange{
      background-color: white;
      position: absolute;
      right: 5px;
      z-index: 10;
      top: 5px;
    }
    .loginButton{
      font-size: 18px;
      font-weight: 400;
    }
    .enterButton{
      background-color: #00c1de;
      font-size: 20px;
      width: 100%;
      cursor: pointer;
      color: white;
    }
  }
</style>
