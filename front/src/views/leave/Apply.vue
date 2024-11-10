<template>
    <div class="main-class">
        <el-steps :active="active" align-center style="margin-top: 20px">
            <el-step title="提醒告知" description="请阅读并同意"></el-step>
            <el-step title="填写请假" description="请详细填写请假信息"></el-step>
            <el-step title="确认请假" description="请确认请假的具体信息"></el-step>
            <el-step title="结果" description="提交结果"></el-step>
        </el-steps>

        <div class="notice" v-show="active === 1">
            <h3 style="text-align: center">提醒告知</h3>
            <ul class="info_list">
                <li>1. 离校去往低风险地区的学生，坚持每日健康打卡且健康码为绿色，方可入校。</li>
                <li>2. 途经中高风险地区或与中高风险地区人员有接触的学生返常后，暂不入校。按属地要求进行隔离并接受核酸检测，经相关部门审核、备案后，方可入校。中、高风险地区名单可到国务院网站查询(网址：<a
                        href="http://bmfw.www.gov.cn/yqfxdjcx/index.html" target="_blank">http://bmfw.www.gov.cn/yqfxdjcx/index.html</a>)。
                </li>
                <li>3. 请假学生本人及同住家人、密切接触者有国（境）外旅居史的，暂缓入校。按属地要求进行隔离并接受核酸检测，经相关部门审核、备案后，方可入校。</li>
                <li>4. 请假学生本人及同住家人、密切接触者出现身体异常情况（发烧、咳嗽等症状）的，暂缓入校。直至排除新冠肺炎可能后，方可申请入校。</li>
                <li>5. 出行期间，请做好个人防护措施。</li>
            </ul>
            <el-checkbox style="margin-left: 108px;margin-top: 10px" v-model="checked">我承诺，本人根据有关要求进行了如实登记，按规定（登记）时间出入校园，如有隐瞒愿意承担相应法律责任。</el-checkbox>
        </div>

        <div class="form" v-show="active === 2">
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="请假类型" prop="leaveType">
                            <el-select
                                    v-model="form.leaveType"
                                    placeholder="请选择请假类型"
                            >
                                <el-option
                                        :key="1"
                                        label="事假"
                                        :value="1"
                                />
                                <el-option
                                        :key="2"
                                        label="病假"
                                        :value="2"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="学生类型" prop="studentType">
                            <el-select
                                    v-model="form.studentType"
                                    placeholder="请选择学生类型"
                            >
                                <el-option
                                        :key="1"
                                        label="本科生"
                                        :value="1"
                                />
                                <el-option
                                        :key="2"
                                        label="硕士生"
                                        :value="2"
                                />
                                <el-option
                                        :key="3"
                                        label="博士生"
                                        :value="3"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="请假时间" prop="time">
                            <el-date-picker
                                    v-model="form.time"
                                    value-format="yyyy-MM-dd"
                                    type="daterange"
                                    style="width: 221px"
                                    range-separator="-"
                                    start-placeholder="开始日期"
                                    end-placeholder="结束日期"
                            ></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="请假天数" prop="day">
                            <el-input-number v-model="form.day" style="width: 221px" controls-position="right" :min="1" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="目的地址" prop="address">
                            <el-input
                                    v-model="form.address"
                                    placeholder="请输入目的地址"
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="交通工具" prop="traffic">
                            <el-input
                                    v-model="form.traffic"
                                    placeholder="请输入交通工具"
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="宿舍门号" prop="dormitory">
                            <el-input
                                    v-model="form.dormitory"
                                    placeholder="请输入宿舍门号"
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="联系号码" prop="phoneNumber">
                            <el-input
                                    v-model="form.phoneNumber"
                                    placeholder="请输入联系号码"
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="课程" prop="clazz">
                            <span slot="label">
                                <el-tooltip content="请假的期间是否有课程" placement="top">
                                <i class="el-icon-question"></i>
                                </el-tooltip>
                                课程
                            </span>
                            <el-select
                                    v-model="form.clazz"
                                    placeholder="请选择课程类型"
                            >
                                <el-option
                                        :key="1"
                                        label="没有"
                                        :value="1"
                                />
                                <el-option
                                        :key="0"
                                        label="有"
                                        :value="0"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="考试" prop="exam">
                            <span slot="label">
                                <el-tooltip content="请假的期间是否有考试" placement="top">
                                <i class="el-icon-question"></i>
                                </el-tooltip>
                                考试
                            </span>
                            <el-select
                                    v-model="form.exam"
                                    placeholder="请选择考试类型"
                            >
                                <el-option
                                        :key="1"
                                        label="没有"
                                        :value="1"
                                />
                                <el-option
                                        :key="0"
                                        label="有"
                                        :value="0"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="请假原因" prop="reason">
                            <el-input
                                    v-model="form.reason"
                                    placeholder="请输入请假原因"
                                    style="width: 581px"
                                    type="textarea"
                            />
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </div>

        <div class="form" v-show="active === 3">
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="请假类型" prop="leaveType">
                            <el-select
                                    v-model="form.leaveType"
                                    disabled
                                    placeholder="请选择请假类型"
                            >
                                <el-option
                                        :key="1"
                                        label="事假"
                                        :value="1"
                                />
                                <el-option
                                        :key="2"
                                        label="病假"
                                        :value="2"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="学生类型" prop="studentType">
                            <el-select
                                    v-model="form.studentType"
                                    disabled
                                    placeholder="请选择学生类型"
                            >
                                <el-option
                                        :key="1"
                                        label="本科生"
                                        :value="1"
                                />
                                <el-option
                                        :key="2"
                                        label="硕士生"
                                        :value="2"
                                />
                                <el-option
                                        :key="3"
                                        label="博士生"
                                        :value="3"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="请假时间" prop="time">
                            <el-date-picker
                                    v-model="form.time"
                                    value-format="yyyy-MM-dd"
                                    type="daterange"
                                    disabled
                                    style="width: 221px"
                                    range-separator="-"
                                    start-placeholder="开始日期"
                                    end-placeholder="结束日期"
                            ></el-date-picker>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="请假天数" prop="day">
                            <el-input-number disabled v-model="form.day" style="width: 221px" controls-position="right" :min="1" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="目的地址" prop="address">
                            <el-input
                                    v-model="form.address"
                                    placeholder="请输入目的地址"
                                    disabled
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="交通工具" prop="traffic">
                            <el-input
                                    v-model="form.traffic"
                                    placeholder="请输入交通工具"
                                    disabled
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="宿舍门号" prop="dormitory">
                            <el-input
                                    v-model="form.dormitory"
                                    placeholder="请输入宿舍门号"
                                    disabled
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="联系号码" prop="phoneNumber">
                            <el-input
                                    v-model="form.phoneNumber"
                                    placeholder="请输入联系号码"
                                    disabled
                                    style="width: 221px"
                            />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="课程" prop="clazz">
                            <span slot="label">
                                <el-tooltip content="请假的期间是否有课程" placement="top">
                                <i class="el-icon-question"></i>
                                </el-tooltip>
                                课程
                            </span>
                            <el-select
                                    v-model="form.clazz"
                                    disabled
                                    placeholder="请选择课程类型"
                            >
                                <el-option
                                        :key="1"
                                        label="没有"
                                        :value="1"
                                />
                                <el-option
                                        :key="0"
                                        label="有"
                                        :value="0"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="考试" prop="exam">
                            <span slot="label">
                                <el-tooltip content="请假的期间是否有考试" placement="top">
                                <i class="el-icon-question"></i>
                                </el-tooltip>
                                考试
                            </span>
                            <el-select
                                    v-model="form.exam"
                                    disabled
                                    placeholder="请选择考试类型"
                            >
                                <el-option
                                        :key="1"
                                        label="没有"
                                        :value="1"
                                />
                                <el-option
                                        :key="0"
                                        label="有"
                                        :value="0"
                                />
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="请假原因" prop="reason">
                            <el-input
                                    v-model="form.reason"
                                    disabled
                                    placeholder="请输入请假原因"
                                    style="width: 581px"
                                    type="textarea"
                            />
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </div>

        <div v-show="active === 4" class="notice">
            <el-result v-if="code === 200" icon="success" title="请假申请成功" subTitle="请等待老师审批">
            </el-result>
            <el-result v-else icon="error" title="请假申请失败" subTitle="请稍后重新尝试">
            </el-result>
        </div>

        <div class="group">
            <el-button style="margin-top: 12px;" v-show="active > 1 && active < 4" @click="before">上一步</el-button>
            <el-button style="margin-top: 12px;" :disabled="!checked" v-show="active < 3" type="primary" @click="next">
                下一步
            </el-button>
            <el-button style="margin-top: 12px;" v-show="active === 3" type="primary" @click="submitForm">提交</el-button>
            <el-button style="margin-top: 12px;" v-show="active === 4" type="primary" @click="addTag">查看请假记录</el-button>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Apply",
        data() {
            return {
                active: 1,
                checked: false,
                form: {
                    leaveType: undefined,
                    studentType: undefined,
                    day: 1,
                    address: undefined,
                    time: [],
                    traffic: undefined,
                    clazz: undefined,
                    exam: undefined,
                    dormitory: undefined,
                    phoneNumber: undefined,
                    reason: undefined
                },
                rules: {
                    leaveType: [{ required: true, message: "请假类型不能为空", trigger: "blur" }],
                    studentType: [{ required: true, message: "学生类型不能为空", trigger: "blur" }],
                    time: [{ required: true, message: "请假时间不能为空", trigger: "blur" }],
                    day: [{ required: true, message: "请假天数不能为空", trigger: "blur" }],
                    address: [{ required: true, message: "目的地址不能为空", trigger: "blur" }],
                    traffic: [{ required: true, message: "交通工具不能为空", trigger: "blur" }],
                    dormitory: [{ required: true, message: "宿舍门号不能为空", trigger: "blur" }],
                    phoneNumber: [{ required: true, message: "联系号码不能为空", trigger: "blur" }],
                    clazz: [{ required: true, message: "课程不能为空", trigger: "blur" }],
                    exam: [{ required: true, message: "考试不能为空", trigger: "blur" }],
                    reason: [{ required: true, message: "请假原因不能为空", trigger: "blur" }],
                },
                code: undefined
            }
        },
        methods: {
            before() {
                if (this.active > 1) {
                    this.active--
                }
            },
            next(){
                if (this.active < 3){
                    if (this.active === 2){
                        this.$refs.form.validate(valid => {
                            if (valid){
                                this.active++
                                this.modal.notify("请确认填写的请假信息")
                            }else {
                                this.modal.notifyWarning("必填项不能为空！")
                            }
                        })
                    }else {
                        this.active++
                    }
                }
            },
            submitForm(){
                this.form.time = this.form.time.join(" 至 ")
                this.$axios.post("/leave/apply", this.form).then(res => {
                    this.code = res.data.code
                    this.modal.notifySuccess(res.data.data)
                    this.active = 4
                })
            },
            addTag() {
                this.$router.push('/leave/record')
                this.$store.commit("addTag", {
                    path: '/leave/record',
                    title: '请假记录',
                    name: 'leave:record:list',
                    parentTitle: '请假管理'
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
    .notice {
        width: 900px;
        margin: 30px auto;
    }

    .form{
        width: 700px;
        margin: 30px auto;
    }

    .group{
        text-align: center;
        margin-top: -10px;
    }

    .info_list {
        list-style: none;

        li {
            line-height: 28px;
            font-size: 17px;
            letter-spacing: 1px;
            text-indent: 33px;
            padding: 2px;

            a {
                color: inherit;
                text-decoration: none;
                border: none;
                -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
                -webkit-user-select: none;
                -moz-user-select: none;
            }

            a:hover {
                color: #315efb;
            }

            a:active, a:visited, a:link, a:focus {
                -webkit-tap-highlight-color: transparent;
                outline: none;
                background: none;
                text-decoration: none;
            }

            ::selection {
                background: #FFF;
                color: #333;
            }

            ::-moz-selection {
                background: #FFF;
                color: #333;
            }
        }
    }
</style>
