<template>
    <div class="container">
        <el-card shadow="always" :body-style="{ padding: '20px' }">
            <template #header>
                <div class="login_title">
                    <span v-if="!isRegister">用户登录</span>
                    <span v-if="isRegister">用户注册</span>

                </div>
            </template>
            <div class="login_img">
                <img src="/logo.jpg" alt="">
                这里请使用你自己的logo
            </div>
            <!-- card body -->
            <div class="login_form">
                <div style="margin: 20px;"></div>
                <el-form  label-width="80px" ref="LRRef" :rules="rules" :model="LRForm">
                    <el-form-item label="用户名" prop="username" >
                        <el-input v-model="LRForm.username" type="text" maxlength="15" show-word-limit clearable placeholder="请输入用户名"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password">
                        <el-input v-model="LRForm.password" type="text" maxlength="15" show-password placeholder="请输入密码"  clearable></el-input>
                    </el-form-item>
                    <template v-if="isRegister">
                        <el-form-item label="重复密码" prop="password_2" >
                            <el-input v-model="LRForm.password_2" type="text" maxlength="15" show-password placeholder="请重复密码"  clearable></el-input>
                        </el-form-item>
                    </template>
                    <el-form-item label="验证码" prop="cap">
                        <div class="login_captcha">
                            <div class="captcha_input">
                                <el-input v-model="LRForm.cap" maxlength="4"></el-input>
                            </div>
                            <div class="captcha_img" @click="refreshCode">
                                <Captcha :identifyCode="identifyCode" :fontSizeMax="20" :fontSizeMin="20"></Captcha>
                            </div>
                        </div>
                    </el-form-item>
                    <div class="login_btn">
                        <el-button type="success" size="large" @click="loginBtn()" v-if="!isRegister">登录</el-button>
                        <!-- <el-button type="info" size="large"  v-if="!isRegister"><router-link to="/register" style="color: white;text-decoration:none">去注册</router-link></el-button> -->
                      <!-- 这里存在问题，没有点击到文字，只点击到了按钮的话，不会发生跳转，所以用router-link来包裹button，同时设置Margin-left -->

                        <router-link to="/register" style="color: white;text-decoration:none;margin-left: 1rem;">
                            <el-button type="info" size="large"  v-if="!isRegister">去注册</el-button>
                        </router-link>

                        <el-button type="success" size="large" @click="registerBtn" v-if="isRegister">提交</el-button>
                        <router-link to="/login" style="color: white;text-decoration:none;margin-left: 1rem;">
                            <el-button type="info" size="large" @click="registerBtn" v-if="isRegister">去登录</el-button>
                        </router-link>

                    </div>
                </el-form>
            </div>
        </el-card>
        
    </div>
</template>
    
<script setup >
    import { reactive,ref,onMounted,defineProps } from 'vue';
    import { ElNotification } from 'element-plus'
    import Captcha from "./Captcha.vue"
    import { login } from "../api/manager"

    var LRRef=ref(null)
    var LRForm=reactive({
        username:"",
        password:"",
        password_2:"",
        cap:""
    })
    // var username=ref("")
    // var password=ref("")
    // var password_2=ref("")
    // var cap=ref("")
    var props=defineProps(["isRegister"])
    var isRegister=props.isRegister
    //-----------------以下为表单验证
    

    var rules=reactive({
        username:[{ required: true, message: '用户名不可为空!',trigger:'blur'}],
        password:[{ required: true, message: '密码不可为空!',trigger:'blur'}],
        password_2:[{ required: true, message: '密码不可为空!',trigger:'blur'}],
        cap:[{ required: true, message: '验证码不可为空!',trigger:'blur'}],
    })
    //这里使用全局store来验证，还是？验证码这里我还真不知道。

    // console.log(loginStore.isRegister)


    var registerBtn=()=>{
        //注册页面，点击提交按钮后的动作
        //这个时候应该校验验证码
    }

    // -------------------------图形验证码
    let identifyCodes = "1234567890"
    let identifyCode = ref('3212')

    const randomNum = (min, max) => {
        return Math.floor(Math.random() * (max - min) + min)
    }
    
    const makeCode = (o, l) => {
        for (let i = 0; i < l; i++) {
            identifyCode.value += o[
                randomNum(0, o.length)
            ];
        }
    }
    
    const refreshCode = () => {
        identifyCode.value = "";
        makeCode(identifyCodes, 4);
        // console.log(identifyCode)
    }
    
    onMounted(() => {
        identifyCode.value = "";
        makeCode(identifyCodes, 4);
    })
    //-----------------------登录模块
    var loginBtn=()=>{
        console.log(LRForm)
        LRRef.value.validate((valid)=>{
            // console.log(valid)
            if(!valid){
                ElNotification({
                    title:"出错了 o_o！",
                    message:"数据校验不成功",
                    type:"error"
                })
            }
            else{
                if(LRForm.cap==identifyCode.value){
                    // console.log("登录")
                    //验证码正确的代码
                    login(LRForm.username,LRForm.password)
                    .then(res=>{
                        console.log(res)
                    })
                    .catch(error=>{
                        console.log(error.response)
                    })
                }
                else{
                    ElNotification({
                        title:"出错了 ~_~！",
                        message:"验证码错误！",
                        type:"error"
                    })
                    //alert("验证码错误")
                    //refreshCode()
                }
            }
        })
    }
</script>
    
<style scoped>
    .container{
        width: 20rem;
        height: 35rem;
        /* position: absolute;
        left: 30%;
        top: 30%; */
        margin: 5rem auto;
        /* border: 1px red solid; */
        /* overflow: hidden; */
        overflow: auto;
    }
    .login_title{
        text-align: center;
        font-size: 2rem;
        font-weight: bold;
    }
    .login_btn{
        display: flex;
        align-items: center;
        /* justify-content: center; */
        justify-content: center;
        
    }
    .login_img{
        /* justify-content: center; */
        /* display: flex;
        align-items: center; */
        /* margin: 1rem auto; */
        margin: auto 2rem;
        /* border: 1px rgb(60, 25, 214) solid; */
        display: flex;
        
        justify-content: center;
    }
    img{
        width: 10rem;
    }
    .login_captcha{
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .captcha_input{
        width: 4rem;
        margin-right: 1rem;
        /* border: 1px rgb(60, 25, 214) solid; */
    }
    .captcha_img{
        width: 3rem;
        height: 2rem;
    }
</style>
