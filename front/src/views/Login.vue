<template>
    <div class="admin-login">
        <div class="login-container">
            <div class="login-left">
                <!-- 放Logo -->
            </div>
            <div class="login-right">
                <el-form
                        :model="loginForm"
                        :rules="loginRules"
                        ref="loginForm"
                        class="login-form"
                        @keyup.enter.native="submitForm('loginForm')"
                >
                    <p style="margin-left: 4px; text-align: left;font-size: 28px;margin-bottom: 30px">疫情防控系统</p>

                    <el-form-item prop="username">
                        <el-input
                                name="username"
                                class="login-input"
                                prefix-icon="el-icon-user"
                                placeholder="用户名/工号"
                                clearable
                                auto-complete="off"
                                v-model="loginForm.username"
                        />
                    </el-form-item>

                    <el-form-item prop="password">
                        <el-input
                                name="password"
                                class="login-input"
                                show-password
                                prefix-icon="el-icon-lock"
                                placeholder="请输入密码"
                                v-model="loginForm.password"
                        />
                    </el-form-item>

                    <el-form-item class="form-captcha" prop="code">
                        <el-input
                                placeholder="验证码"
                                name="code"
                                prefix-icon="el-icon-c-scale-to-original"
                                class="login-input"
                                style="width: 52%;float: left"
                                type="text"
                                v-model="loginForm.code"
                        />
                        <img
                                style="width: 162px; height: 42px;float: left;margin-left: 22px"
                                :src="captchaImg"
                                @click="getCaptchaImg"
                        />
                    </el-form-item>

                    <el-button
                            style="width: 100%;"
                            type="primary"
                            :loading="btnLoading"
                            @click="submitForm('loginForm')"
                    >
                        <span v-if="!btnLoading">登 录</span>
                        <span v-else>登 录 中...</span>
                    </el-button>
                    <div style="float: right;margin-top: 15px;margin-bottom: -21px" v-if="register">
                        没有账号？<router-link class="link-type" :to="'/register'">立即注册</router-link>
                    </div>
                </el-form>
            </div>
        </div>
    </div>
</template>

<script>
    import qs from 'qs'

    export default {
        name: "Login",
        data() {
            return {
                loginRules: {
                    username: [{
                        required: true,
                        message: '请输入用户名',
                        trigger: 'blur'
                    }],
                    password: [{
                        required: true,
                        message: '请输入密码',
                        trigger: 'blur'
                    }],
                    code: [{
                        required: true,
                        message: '请输入验证码',
                        trigger: 'blur'
                    },
                        {
                            min: 5,
                            max: 5,
                            message: '长度为 5 个字符',
                            trigger: 'blur'
                        }
                    ]
                },
                loginForm: {
                    username: 'admin',
                    password: '123456',
                    code: '',
                    key: ''
                },
                captchaImg: null,
                btnLoading: false,
                register: true
            }
        },
        methods: {
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.btnLoading  = true
                        this.$axios.post('/login?' + qs.stringify(this.loginForm)).then(res => {
                            const jwt = res.headers['authorization']
                            this.$store.commit('SET_TOKEN', jwt)
                            this.$axios.get("/userInfo").then(res => {
                                this.$store.commit('SET_USER_INFO', res.data.data.user)
                            })
                            this.$router.push({path:"/"})
                            let message
                            if (new Date().getHours() < 12){
                                message = "上午好！" + this.loginForm.username
                            }else {
                                message = "下午好！" + this.loginForm.username
                            }
                            this.modal.notifySuccess(message)
                        }).catch(() => {
                            this.btnLoading = false
                        })
                    } else {
                        this.$message.error('数据格式不正确，请重新输入！');
                        return false;
                    }
                });
            },
            getCaptchaImg(){
                this.$axios.get('/captcha').then(res => {
                    this.loginForm.key = res.data.data.key
                    this.captchaImg = res.data.data.base64
                    this.loginForm.code = ''
                })
            }
        },
        created() {
            this.getCaptchaImg()
            let tag = {
                path: '/',
                title: '首页',
                name: 'index',
                parentTitle: "首页"
            }
            this.$store.commit('SET_TOKEN', undefined)
            localStorage.setItem('currentTag', JSON.stringify(tag))
        }
    }
</script>

<style lang="scss" scoped>
    $bg-input: #f1f2f3;

    .admin-login {
        position: relative;
        height: 100vh;
        padding: 0 12rem;
        overflow: hidden;
        background-image: url("../assets/bg.jpg");
        background-size: cover;
    }

    .login-container {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: space-around;
        height: 100%;
    }

    .login-left {
        width: 35%;
        min-width: 300px;
        max-width: 600px;
        height: 500px;
        background-size: 100%;

        p {
            font-size: 47px;
        }

        div {
            font-size: 23px;
        }
    }

    .login-right {
        display: flex;
        justify-content: flex-end;
        width: 50%;
        height: 500px;
    }

    .login-form {
        align-self: center;
        width: 385px;
        padding: 0 2rem 3rem;
        background: white;
        border-radius: 10px;
        box-shadow: 3px 3px 5px;
    }

    $input-height: 44px;

    .login-input {
        height: $input-height;
        font-size: 17px;
        line-height: $input-height;
    }

    .link-type,
    .link-type:focus {
        color: #337ab7;
        cursor: pointer;

        &:hover {
            color: rgb(32, 160, 255);
        }
    }

</style>
