<template>
    <div class="admin-login">
        <div class="login-container">
            <div class="login-left">
                <!-- 放Logo -->
            </div>
            <div class="login-right">
                <el-form
                        :model="registerForm"
                        :rules="registerRules"
                        ref="registerForm"
                        class="login-form"
                        @keyup.enter.native="submitForm('registerForm')"
                >
                    <p style="margin-left: 4px; text-align: left;font-size: 28px;margin-bottom: 30px">疫情防控系统-注册</p>

                    <el-row>

                        <el-col :span="24">
                            <el-form-item prop="username">
                                <el-input
                                        name="username"
                                        class="login-input"
                                        prefix-icon="el-icon-user"
                                        placeholder="请输入学号/职工号"
                                        auto-complete="off"
                                        v-model="registerForm.username"
                                />
                            </el-form-item>
                        </el-col>

                        <el-col :span="24">
                            <el-form-item prop="password">
                                <el-input
                                        name="password"
                                        class="login-input"
                                        show-password
                                        prefix-icon="el-icon-lock"
                                        placeholder="请输入密码"
                                        v-model="registerForm.password"
                                />
                            </el-form-item>
                        </el-col>

                        <el-col :span="24">
                            <el-form-item class="form-captcha" prop="code">
                                <el-input
                                        placeholder="验证码"
                                        name="code"
                                        prefix-icon="el-icon-c-scale-to-original"
                                        class="login-input"
                                        style="width: 52%;float: left"
                                        type="text"
                                        v-model="registerForm.code"
                                />
                                <img
                                        style="width: 162px; height: 42px;float: left;margin-left: 22px"
                                        :src="captchaImg"
                                        @click="getCaptchaImg"
                                />
                            </el-form-item>
                        </el-col>

                        <el-col :span="12">
                            <el-form-item prop="roleType">
                                <el-select
                                        v-model="registerForm.roleType"
                                        placeholder="角色类型"
                                        style="width: 90%"
                                >
                                    <el-option
                                            :key="2"
                                            label="学生用户"
                                            :value="2"
                                    />
                                    <el-option
                                            :key="1"
                                            label="教师用户"
                                            :value="1"
                                    />
                                    <el-option
                                            :key="0"
                                            label="后勤用户"
                                            :value="0"
                                    />
                                </el-select>
                            </el-form-item>
                        </el-col>

                        <el-col :span="12">
                            <el-form-item prop="registerCode" style="margin-top: -1.6px;">
                                <el-input
                                        name="registerCode"
                                        class="login-input"
                                        style="width: 100%;"
                                        placeholder="请输入注册码"
                                        v-model="registerForm.registerCode"
                                />
                            </el-form-item>
                        </el-col>

                        <el-col :span="12">
                            <el-form-item prop="deptId">
                                <el-select v-model="registerForm.deptId" placeholder="请选择归属部门" style="width: 90%">
                                    <template v-for="item in deptOptions">
                                        <el-option :label="item.deptName" disabled :value="item.deptId"></el-option>
                                        <template v-for="child in item.children">
                                            <el-option :label="child.deptName" :value="child.deptId">
                                                <span>{{ "- " + child.deptName }}</span>
                                            </el-option>
                                        </template>
                                    </template>
                                </el-select>
                            </el-form-item>
                        </el-col>

                        <el-col :span="12">
                            <el-form-item prop="phoneNumber">
                                <el-input
                                        name="registerCode"
                                        class="login-input"
                                        style="margin-top: -1.6px;"
                                        placeholder="请输入手机号码"
                                        v-model="registerForm.phoneNumber"
                                />
                            </el-form-item>
                        </el-col>

                    </el-row>

                    <el-button
                            style="width: 100%;"
                            type="primary"
                            :loading="btnLoading"
                            @click="submitForm"
                    >
                        <span v-if="!btnLoading">注 册</span>
                        <span v-else>注 册 中...</span>
                    </el-button>
                    <div style="float: right;margin-top: 15px;margin-bottom: -21px">
                        有账号了？<router-link class="link-type" :to="'/login'">立即登录</router-link>
                    </div>
                </el-form>
            </div>
        </div>
    </div>
</template>

<script>
    import qs from "qs";
    import router from "../router";

    export default {
        name: "Register",
        data() {
            return {
                registerRules: {
                    username: [{
                        required: true,
                        message: '请输入学号/职工号',
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
                    ],
                    roleType: [
                        {
                            required: true,
                            message: '请选择角色类型',
                            trigger: 'blur'
                        }
                    ],
                    registerCode: [
                        {
                            required: true,
                            message: '请输入注册码',
                            trigger: 'blur'
                        }
                    ],
                    phoneNumber: [
                        { required: true, message: "手机号不能为空", trigger: "blur" },
                        {
                            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
                            message: "请输入正确的手机号码",
                            trigger: "blur"
                        }
                    ],
                },
                registerForm: {
                    username: undefined,
                    password: undefined,
                    roleType: undefined,
                    registerCode: undefined,
                    deptId: undefined,
                    phoneNumber: undefined,
                    code: undefined,
                    key: undefined
                },
                deptOptions: undefined,
                captchaImg: null,
                btnLoading: false,
            }
        },
        methods: {
            submitForm() {
                this.$refs['registerForm'].validate((valid) => {
                    if (valid) {
                        this.$axios.post("/register?" + qs.stringify(this.registerForm)).then(res => {
                            this.btnLoading = true
                            if (res.data.code === 200){
                                this.modal.notifySuccess("注册成功，即将转往登陆界面")
                                setTimeout(() => {
                                    router.push({path: '/login'})
                                }, 3000)
                            }
                            this.btnLoading = false
                        })
                    } else {
                        this.$message.error('数据格式不正确，请重新输入！');
                        return false;
                    }
                    this.btnLoading = false
                })
            },
            getCaptchaImg(){
                this.$axios.get('/captcha').then(res => {
                    this.registerForm.key = res.data.data.key
                    this.captchaImg = res.data.data.base64
                    this.registerForm.code = ''
                })
            },
            getDeptList(){
                this.$axios.get("/register/deptList").then(res => {
                    this.deptOptions = res.data.data[0].children
                })
            }
        },
        created() {
            this.getCaptchaImg()
            this.$store.commit('SET_TOKEN', undefined)
            this.getDeptList()
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
