<template>
    <div class="main-class">
        <el-form v-permission="['ROLE_teacher']" :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
            <el-form-item label="用户名称" prop="username">
                <el-input
                        v-model="queryParams.username"
                        placeholder="请输入用户名称"
                        clearable
                        size="small"
                        style="width: 138px"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="归属部门" prop="deptId">
                <el-select v-model="queryParams.deptId" clearable placeholder="请选择归属部门" size="small" style="width: 160px">
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
            <el-form-item label="请假类型" prop="diagnosis">
                <el-select
                        v-model="queryParams.diagnosis"
                        placeholder="请假类型"
                        clearable
                        size="small"
                        style="width: 100px"
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
            <el-form-item label="请假状态" prop="diagnosis">
                <el-select
                        v-model="queryParams.status"
                        placeholder="请假类型"
                        clearable
                        size="small"
                        style="width: 120px"
                >
                    <el-option
                            :key="1"
                            label="待审核"
                            :value="1"
                    />
                    <el-option
                            :key="2"
                            label="审核通过"
                            :value="2"
                    />
                    <el-option
                            :key="3"
                            label="审核不通过"
                            :value="3"
                    />
                </el-select>
            </el-form-item>
            <el-form-item label="提交时间" prop="dateRange">
                <el-date-picker
                        v-model="dateRange"
                        size="small"
                        style="width: 211px"
                        value-format="yyyy-MM-dd"
                        type="daterange"
                        range-separator="-"
                        start-placeholder="开始日期"
                        end-placeholder="结束日期"
                ></el-date-picker>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
            </el-form-item>
        </el-form>

        <!--数据表格-->
        <el-table v-loading="loading" :data="dataList">
            <el-table-column label="编号" align="center" width="80" prop="id" />
            <el-table-column label="账号" align="center" prop="username" />
            <el-table-column label="姓名" align="center" prop="nickname" />
            <el-table-column prop="leaveType" label="请假类型" align="center">
                <template slot-scope="scope">
                    <span v-if="scope.row.leaveType === 1">事假</span>
                    <span v-else-if="scope.row.leaveType === 2">病假</span>
                    <span v-else>其他</span>
                </template>
            </el-table-column>
            <el-table-column prop="studentType" label="学生类型" align="center">
                <template slot-scope="scope">
                    <span v-if="scope.row.studentType === 1">本科生</span>
                    <span v-else-if="scope.row.studentType === 2">硕士生</span>
                    <span v-else>博士生</span>
                </template>
            </el-table-column>
            <el-table-column label="请假时间" align="center" prop="time" width="200" />
            <el-table-column label="目的地" align="center" prop="address" />
            <el-table-column label="宿舍门号" align="center" prop="dormitory" />
            <el-table-column prop="status" label="请假状态" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.status === 3" type="danger">审核不通过</el-tag>
                    <el-tag size="small" v-else-if="scope.row.status === 2" type="success">审核通过</el-tag>
                    <el-tag size="small" v-else-if="scope.row.status === 1" type="warning">待审核</el-tag>
                    <el-tag size="small" v-else-if="scope.row.status === 0" type="info">撤销</el-tag>
                </template>
            </el-table-column>
            <el-table-column
                    label="操作"
                    align="center"
                    width="200"
                    class-name="small-padding fixed-width"
            >
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-refresh-left"
                            v-permission="['ROLE_student']"
                            v-show="scope.row.status === 1"
                            v-is-admin
                            @click="update(JSON.stringify(scope.row), '撤销')"
                    >撤销</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-refresh-right"
                            v-show="scope.row.status === 0"
                            v-permission="['ROLE_student']"
                            v-is-admin
                            @click="update(JSON.stringify(scope.row), '提交')"
                    >提交</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-view"
                            @click="handleView(scope.row)"
                    >详细</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-edit"
                            v-show="scope.row.status === 0"
                            @click="update(JSON.stringify(scope.row), '修改')"
                            v-permission="['ROLE_student']"
                    >修改</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-edit"
                            @click="examine(JSON.stringify(scope.row))"
                            v-show="scope.row.status === 1"
                            v-permission="['ROLE_teacher']"
                    >审批</el-button>
                </template>
            </el-table-column>
        </el-table>
        <Pagination
                v-show="total>0"
                :total="total"
                :page.sync="queryParams.pageNum"
                :limit.sync="queryParams.pageSize"
                @pagination="getList"
        />
        <el-dialog title="请假记录详细" :visible.sync="recordOpen" width="700px" append-to-body>
            <el-form ref="form" :model="record" label-width="100px" size="mini">
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="用户名：">{{ record.username }}</el-form-item>
                        <el-form-item label="学生类型：">
                            <span v-if="record.studentType === 1">本科生</span>
                            <span v-else-if="record.studentType === 2">硕士生</span>
                            <span v-else>博士生</span>
                        </el-form-item>
                        <el-form-item label="请假时间：">{{ record.time }}</el-form-item>
                        <el-form-item label="目的地：">{{ record.address }}</el-form-item>
                        <el-form-item label="宿舍门号：">{{ record.dormitory }}</el-form-item>
                        <el-form-item label="有无课程：">
                            <span v-if="record.clazz === 1">没有</span>
                            <span v-else-if="record.clazz === 0">有</span>
                        </el-form-item>
                        <el-form-item label="申请时间：">{{ record.createTime }}</el-form-item>
                        <el-form-item label="请假状态：">
                            <span v-if="record.status === 0">撤销</span>
                            <span v-else-if="record.exam === 1">待审核</span>
                            <span v-else-if="record.exam === 2">审核通过</span>
                            <span v-else>审核不通过</span>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="学生姓名：">{{ record.nickname }}</el-form-item>
                        <el-form-item label="请假类型：">
                            <span v-if="record.studentType === 1">事假</span>
                            <span v-else-if="record.studentType === 2">病假</span>
                            <span v-else>博士生</span>
                        </el-form-item>
                        <el-form-item label="请假天数：">{{ record.day }}</el-form-item>
                        <el-form-item label="交通工具：">{{ record.traffic }}</el-form-item>
                        <el-form-item label="手机号码：">{{ record.phoneNumber }}</el-form-item>
                        <el-form-item label="有无考试：">
                            <span v-if="record.exam === 1">没有</span>
                            <span v-else-if="record.exam === 0">有</span>
                        </el-form-item>
                        <el-form-item label="修改时间：">{{ record.updateTime }}</el-form-item>
                        <el-form-item label="审核意见：">
                            <span v-if="record.opinion">{{ record.opinion }}</span>
                            <span v-else>暂无</span>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="请假理由：">
                            <span v-if="record.reason">{{ record.reason }}</span>
                            <span v-else>暂无</span>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="recordOpen = false">关 闭</el-button>
            </div>
        </el-dialog>
        <el-dialog title="修改请假记录" :visible.sync="updateOpen" width="700px" append-to-body>
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
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitUpdate">确 定</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
        <el-dialog title="审核请假记录" :visible.sync="examineOpen" width="700px" append-to-body>
            <el-form ref="form" :model="examineForm" :rules="examineRules" label-width="80px">
                <el-form-item label="审核状态">
                    <el-radio-group v-model="examineForm.status">
                        <el-radio :label=2><span style="font-weight: normal">通过</span></el-radio>
                        <el-radio :label=3><span style="font-weight: normal">不通过</span></el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="审批意见" prop="reason">
                    <el-input
                            v-model="examineForm.opinion"
                            placeholder="请输入审批意见"
                            type="textarea"
                    />
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitExamine">确 定</el-button>
                <el-button @click="examineOpen = false">取 消</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Record",
        data(){
            return{
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    username: undefined,
                    deptId: undefined,
                    leaveType: undefined,
                    status: undefined
                },
                deptOptions: [],
                dateRange: [],
                loading: false,
                dataList: [],
                record: {},
                recordOpen: false,
                total: 0,
                form: {},
                examineForm: {},
                examineOpen: false,
                updateOpen: false,
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
                examineRules: {

                }
            }
        },
        methods: {
            resetForm(formName) {
                if (this.$refs[formName]) {
                    this.$refs[formName].resetFields();
                }
            },
            reset() {
                this.form = {}
                this.resetForm("form")
            },
            cancel() {
                this.updateOpen = false
                this.reset()
            },
            handleQuery() {
                this.queryParams.pageNum = 1
                this.getList()
            },
            getList(){
                this.loading = true
                this.$axios.get("/leave/apply/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        deptId: this.queryParams.deptId,
                        username: this.queryParams.username,
                        leaveType: this.queryParams.leaveType,
                        status: this.queryParams.status,
                        start: this.dateRange[0] ? this.dateRange[0] : undefined,
                        end: this.dateRange[1] ? this.dateRange[1] :undefined
                    }}).then(res => {
                    this.dataList = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            },
            getDeptList(){
                this.$axios.get("/sys/dept/list/true").then(res => {
                    this.deptOptions = res.data.data[0].children
                })
            },
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    username: undefined,
                    deptId: undefined,
                    leaveType: undefined,
                    status: undefined
                }
                this.dateRange = []
                this.resetForm("queryForm");
                this.handleQuery();
            },
            handleView(row) {
                this.recordOpen = true
                this.record = row
            },
            update(row, type){
                this.form = JSON.parse(row)
                this.form.time = this.form.time.split(" 至 ")
                console.log(this.form)
                if (type === '撤销'){
                    this.form.status = 0
                    this.modal.confirm('确定要撤销请假吗？').then(function () {
                    }).then(() => {
                        this.updateData()
                    }).catch(()=>{})
                }else if (type === '提交'){
                    this.form.status = 1
                    this.modal.confirm('确定要提交请假吗？').then(function () {
                    }).then(() => {
                        this.updateData()
                    }).catch(()=>{})
                }else if (type === '修改'){
                    this.updateOpen = true
                }
            },
            updateData(){
                this.form.time = this.form.time.join(" 至 ")
                this.$axios.put("/leave/apply", this.form).then(res => {
                    this.modal.notifySuccess(res.data.data)
                    this.getList()
                })
            },
            submitUpdate(){
                this.updateData()
                this.updateOpen = false
            },
            examine(row){
                this.examineForm = JSON.parse(row)
                this.examineForm.status = 2
                this.examineOpen = true
            },
            submitExamine(){
                this.$axios.put("/leave/apply", this.examineForm).then(res => {
                    this.modal.notifySuccess(res.data.data)
                    this.getList()
                    this.examineOpen = false
                })
            }
        },
        created() {
            this.getList()
            this.getDeptList()
        }
    }
</script>

<style scoped>

</style>
