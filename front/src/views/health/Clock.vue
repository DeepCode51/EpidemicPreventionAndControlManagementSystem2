<template>
    <div class="main-class">
        <el-steps :active="active" align-center style="margin-top: 20px">
            <el-step title="填写" description="请填写相关信息"></el-step>
            <el-step title="确认" description="确认信息无误"></el-step>
            <el-step title="完成" description="完成健康打卡"></el-step>
        </el-steps>
        <div class="content">
            <div v-show="active === 1" style="text-align: center">
                <el-form ref="form" :model="form" :rules="rules" style="display: inline-block">
                    <el-form-item label="1. 目前的健康状况：" prop="healthType">
                        <el-radio-group v-model="form.healthType">
                            <el-radio :label=2><span>正常</span></el-radio>
                            <el-radio :label=1><span>有发热咳嗽</span></el-radio>
                            <el-radio :label=0><span>其他不正常状况</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="2. 当前体温：" prop="temperature">
                        <el-input v-model="form.temperature" placeholder="请输入当前体温" clearable />
                    </el-form-item>
                    <el-form-item label="3. 是否中高风险地区停留或接触中高风险地区人员：" prop="middleHigh">
                        <el-radio-group v-model="form.middleHigh">
                            <el-radio :label=1><span>否</span></el-radio>
                            <el-radio :label=0><span>是</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="4. 是否接触疑似或者确诊的新冠病毒肺炎患者患者：" prop="diagnosis" style="width: 460px">
                        <el-radio-group v-model="form.diagnosis">
                            <el-radio :label=1><span>否</span></el-radio>
                            <el-radio :label=0><span>是</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="5. 近14日有无与境外（包括港澳台）回国人员接触：" prop="returnInfo">
                        <el-radio-group v-model="form.returnInfo">
                            <el-radio :label=1><span>否</span></el-radio>
                            <el-radio :label=0><span>是</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="6. 请选择打卡时所在的位置信息：" prop="address">
                        <el-cascader ref="ownArea" @change="selectArea" :props="{value:'label', label: 'label', children: 'children'}" :options="arercity" v-model="form.address"
                                     props.expandTrigger="hover" clearable placeholder="请选择地区位置"></el-cascader>

                    </el-form-item>
                </el-form>
            </div>
            <div v-show="active === 2" style="text-align: center">
                <el-form ref="form" :model="form" :rules="rules" style="display: inline-block">
                    <el-form-item label="1. 目前的健康状况：" prop="healthType">
                        <el-radio-group v-model="form.healthType" disabled>
                            <el-radio :label=2><span>正常</span></el-radio>
                            <el-radio :label=1><span>有发热咳嗽</span></el-radio>
                            <el-radio :label=0><span>其他不正常状况</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="2. 当前体温：" prop="temperature">
                        <el-input v-model="form.temperature" placeholder="请输入当前体温" clearable disabled />
                    </el-form-item>
                    <el-form-item label="3. 是否中高风险地区停留或接触中高风险地区人员：" prop="middleHigh">
                        <el-radio-group v-model="form.middleHigh" disabled>
                            <el-radio :label=1><span>否</span></el-radio>
                            <el-radio :label=0><span>是</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="4. 是否接触疑似或者确诊的新冠病毒肺炎患者患者：" prop="diagnosis" style="width: 460px">
                        <el-radio-group v-model="form.diagnosis" disabled>
                            <el-radio :label=1><span>否</span></el-radio>
                            <el-radio :label=0><span>是</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="5. 近14日有无与境外（包括港澳台）回国人员接触：" prop="returnInfo">
                        <el-radio-group v-model="form.returnInfo" disabled>
                            <el-radio :label=1><span>否</span></el-radio>
                            <el-radio :label=0><span>是</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="6. 请选择打卡时所在的位置信息：" prop="address">
                        <el-cascader ref="ownArea" @change="selectArea" disabled :props="{value:'label', label: 'label', children: 'children'}" :options="arercity" v-model="form.address"
                                     props.expandTrigger="hover" clearable placeholder="请选择地区位置"></el-cascader>

                    </el-form-item>
                </el-form>
            </div>
            <div v-show="active === 3">
                <el-result v-if="code === 200" icon="success" title="打卡成功" subTitle="请根据提示进行操作">
                </el-result>
                <el-result v-else icon="error" title="打卡失败" subTitle="请根据提示进行操作">
                </el-result>
            </div>
        </div>
        <div class="group">
            <el-button style="margin-top: 12px;" v-show="active > 1 && active < 3" @click="before">上一步</el-button>
            <el-button style="margin-top: 12px;" v-show="active < 2" type="primary" @click="next">下一步</el-button>
            <el-button style="margin-top: 12px;" v-show="active === 2" type="primary" @click="submitForm">提交</el-button>
            <el-button style="margin-top: 12px;" v-show="active === 3" type="primary" @click="addTag">查看打卡记录</el-button>
        </div>
    </div>
</template>

<script>
    import areaDict from "../../plugins/area"
    export default {
        name: "Clock",
        data() {
            let validateInteger = (rule, value, callback) => {
                if (!value) {
                    return callback(new Error("体温信息不能为空"));
                } else if (parseInt(value) > 50 || parseInt(value) <= 35) {
                    return callback(new Error(`请输入正常体温范围的值`));
                } else {
                    return callback()
                }
            };
            return {
                active: 1,
                form: {
                    healthType: 2,
                    temperature: undefined,
                    middleHigh: 1,
                    diagnosis: 1,
                    returnInfo: 1,
                    address: [],
                    username: undefined,
                    deptId: undefined
                },
                rules: {
                    healthType: [
                        { required: true, message: "健康状况不能为空", trigger: "blur" }
                    ],
                    temperature: [
                        { required: true, message: "打卡信息不能为空", trigger: "blur" },
                        { required: true, validator: validateInteger, trigger: "blur" }
                    ],
                    middleHigh: [
                        { required: true, message: "健康信息不能为空", trigger: "blur" }
                    ],
                    diagnosis: [
                        { required: true, message: "健康信息不能为空", trigger: "blur" }
                    ],
                    returnInfo: [
                        { required: true, message: "健康信息不能为空", trigger: "blur" }
                    ],
                    address: [
                        { required: true, message: "打卡地址不能为空", trigger: "blur" }
                    ]
                },
                arercity: areaDict,
                code: undefined
            }
        },
        methods: {
            next() {
                this.$refs.form.validate(valid => {
                    if (valid){
                        if (this.active < 3){
                            this.active++
                            if(this.active === 2){
                                this.modal.notify("请确认填写信息无误")
                            }
                        }
                    }else {
                        this.$message.error('请正确输入打卡信息！');
                        return false
                    }
                })
            },
            before(){
                if (this.active > 1){
                    this.active--
                }
            },
            selectArea () {
                this.$refs.ownArea.dropDownVisible = false
            },
            submitForm (){
                this.$refs.form.validate(valid => {
                  if (valid){
                      this.getUserInfo()
                      if (Array.isArray(this.form.address)){
                          this.form.address = this.form.address.join("-")
                      }
                      this.$axios.post("/health/clock", this.form).then(res => {
                          this.code = res.data.code
                          this.modal.notifySuccess(res.data.data)
                          this.active = 3
                      })
                  }
                })
            },
            getUserInfo(){
                const user = JSON.parse(localStorage.getItem("userInfo"));
                this.form.username = user.username
                this.form.deptId = user.deptId
            },
            addTag() {
                this.$router.push('/health/clockInfo')
                this.$store.commit("addTag", {
                    path: '/health/clockInfo',
                    title: '打卡信息',
                    name: 'health:clock:list',
                    parentTitle: '健康管理'
                })
            }
        },
        created() {
            this.$axios("/health/clock").then(res => {
                if (res.data.data !== 'allow'){
                    this.active = 3
                    this.code = 200
                    this.modal.notifyWarning(res.data.data)
                }
            })
        }
    }
</script>

<style lang="scss" scoped>
    .group{
        text-align: center;
    }
    .content{
        margin: 25px auto;
    }
    .el-form-item {
        margin-bottom: 3px !important;
    }
    // 级联下拉框单选 文字选中样式
    .selectArea {
        .el-radio {
            width: 100%;
            height: 100%;
            z-index: 10;
            position: absolute;
        }
        .is-disabled {
            cursor: not-allowed;
        }
        .el-radio__input {
            visibility: hidden;
        }
    }
</style>
