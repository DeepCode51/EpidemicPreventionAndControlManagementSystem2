<template>
    <div class="main-class">
        <el-steps :active="active" simple>
            <el-step title="上传报告" icon="el-icon-edit"></el-step>
            <el-step title="确认报告" icon="el-icon-upload"></el-step>
            <el-step title="操作结果" icon="el-icon-picture"></el-step>
        </el-steps>
        <div class="content">
            <div v-show="active === 1" style="text-align: center">
                <el-form ref="form" :model="form" style="display: inline-block">
                    <el-row>
                        <el-col :span="12">
                            <el-form-item label="1. 当前健康码" prop="img">
                                <el-upload
                                        class="avatar-uploader"
                                        action="#"
                                        multiple
                                        :show-file-list="false"
                                        :http-request="requestUpload"
                                        :before-upload="beforeAvatarUpload1">
                                    <img v-if="img1" :src="img1" class="avatar" alt="">
                                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                                </el-upload>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="2. 当前行程码" prop="img">
                                <el-upload
                                        class="avatar-uploader"
                                        action="#"
                                        multiple
                                        :show-file-list="false"
                                        :http-request="requestUpload"
                                        :before-upload="beforeAvatarUpload2">
                                    <img v-if="img2" :src="img2" class="avatar" alt="">
                                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                                </el-upload>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="3. 核酸检测报告" prop="img">
                                <el-upload
                                        class="avatar-uploader"
                                        action="#"
                                        multiple
                                        :show-file-list="false"
                                        :http-request="requestUpload"
                                        :before-upload="beforeAvatarUpload3">
                                    <img v-if="img3" :src="img3" class="avatar" alt="">
                                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                                </el-upload>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item label="5. 返校情况" prop="type">
                        <el-radio v-model="form.type" :label="1"><span style="font-weight: normal">假期或请假返校</span>
                        </el-radio>
                        <el-radio v-model="form.type" :label="0"><span style="font-weight: normal">其他</span></el-radio>
                    </el-form-item>
                    <el-checkbox v-model="checked">我承诺以上材料真实有效，否则对产生的任何后果负责</el-checkbox>
                </el-form>
            </div>
            <div v-show="active === 2" style="text-align: center">
                <el-form ref="form" :model="form" style="display: inline-block">
                    <el-row>
                        <el-col :span="12">
                            <el-form-item label="1. 当前健康码">
                                <el-image
                                        class="avatar"
                                        :src="img1"
                                        :preview-src-list="[img1]">
                                </el-image>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="2. 当前行程码">
                                <el-image
                                        class="avatar"
                                        :src="img2"
                                        :preview-src-list="[img2]">
                                </el-image>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="3. 核酸检测报告">
                                <el-image
                                        class="avatar"
                                        :src="img3"
                                        :preview-src-list="[img3]">
                                </el-image>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item label="5. 返校情况" prop="type">
                        <el-radio v-model="form.type" disabled :label="1"><span style="font-weight: normal">假期或请假返校</span>
                        </el-radio>
                        <el-radio v-model="form.type" disabled :label="0"><span style="font-weight: normal">其他</span></el-radio>
                    </el-form-item>
                </el-form>
            </div>
            <div v-show="active === 3">
                <el-result v-if="code === 200" icon="success" title="上报成功" subTitle="请根据提示进行操作">
                </el-result>
                <el-result v-else icon="error" title="上报失败" subTitle="请根据提示进行操作">
                </el-result>
            </div>
        </div>
        <div class="group">
            <el-button style="margin-top: 12px;" v-show="active > 1 && active < 3" @click="before">上一步</el-button>
            <el-button style="margin-top: 12px;" :disabled="!checked" v-show="active < 2" type="primary" @click="next">
                下一步
            </el-button>
            <el-button style="margin-top: 12px;" v-show="active === 2" type="primary" @click="submitForm">提交</el-button>
            <el-button style="margin-top: 12px;" v-show="active === 3" type="primary" @click="addTag">查看打卡记录</el-button>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Report",
        data() {
            return {
                active: 1,
                form: {
                    type: 1
                },
                file1: undefined,
                file2: undefined,
                file3: undefined,
                img1: undefined,
                img2: undefined,
                img3: undefined,
                checked: false,
                code: undefined
            }
        },
        methods: {
            requestUpload() {
            },
            beforeAvatarUpload1(file) {
                if (file.type.indexOf("image/") === -1) {
                    this.modal.msgError("文件格式错误，请上传图片类型,如：JPG，PNG后缀的文件。")
                } else {
                    const reader = new FileReader()
                    reader.readAsDataURL(file)
                    reader.onload = () => {
                        this.img1 = reader.result
                    }
                    this.file1 = file
                }
            },
            beforeAvatarUpload2(file) {
                if (file.type.indexOf("image/") === -1) {
                    this.modal.msgError("文件格式错误，请上传图片类型,如：JPG，PNG后缀的文件。")
                } else {
                    const reader = new FileReader()
                    reader.readAsDataURL(file)
                    reader.onload = () => {
                        this.img2 = reader.result
                    }
                    this.file2 = file
                }
            },
            beforeAvatarUpload3(file) {
                if (file.type.indexOf("image/") === -1) {
                    this.modal.msgError("文件格式错误，请上传图片类型,如：JPG，PNG后缀的文件。")
                } else {
                    const reader = new FileReader()
                    reader.readAsDataURL(file)
                    reader.onload = () => {
                        this.img3 = reader.result
                    }
                    this.file3 = file
                }
            },
            next() {
                if (this.active < 3) {
                    if (this.file1 === undefined || this.file2 === undefined || this.file3 === undefined){
                        this.modal.notifyError("所有图片都需要上传！")
                        this.active = 1
                        return
                    }
                    this.active++
                    if (this.active === 2) {
                        this.modal.notify("请确认图片，点击图片查看高清大图")
                    }
                }
            },
            before() {
                if (this.active > 1) {
                    this.active--
                }
            },
            submitForm(){
                let formData = new FormData()
                formData.append("file1", this.file1)
                formData.append("file2", this.file2)
                formData.append("file3", this.file3)
                formData.append("type", this.form.type)
                this.$axios.post("/health/report", formData).then(res => {
                    this.code = res.data.code
                    this.modal.notifySuccess(res.data.data)
                    this.active = 3
                })
            },
            addTag() {
                this.$router.push('/health/reportInfo')
                this.$store.commit("addTag", {
                    path: '/health/reportInfo',
                    title: '报告信息',
                    name: 'health:report:list',
                    parentTitle: '健康管理'
                })
            }
        },
        created() {
            this.$axios("/health/report").then(res => {
                if (res.data.data !== 'allow'){
                    this.active = 3
                    this.code = 200
                    this.modal.notifyWarning(res.data.data)
                }
            })
        }
    }
</script>


<style lang="scss">
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 10px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 110px;
        height: 110px;
        line-height: 110px;
        text-align: center;
    }
    .avatar {
        width: 110px;
        height: 110px;
        display: block;
    }
</style>

<style lang="scss" scoped>
    .content {
        margin: 15px auto;
    }
    .el-form-item {
        padding: 5px 25px !important;
        margin-bottom: 10px !important;
    }
    .group{
        text-align: center !important;
    }
</style>
