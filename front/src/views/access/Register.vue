<template>
    <div class="main-class">
        <el-row :gutter="20">
            <el-col :span="7">
                <el-card shadow="hover">
                    <el-form v-permission="['ROLE_service']" ref="form" :rules="rules" :model="form" label-width="80px">
                        <el-form-item label="人员姓名" autocomplete="off"
                                      prop="name">
                            <el-input v-model="form.name" placeholder="请输入姓名" maxlength="30"/>
                        </el-form-item>
                        <el-form-item label="联系电话" autocomplete="off"
                                      prop="phone">
                            <el-input v-model="form.phone" placeholder="请输入联系人电话" />
                        </el-form-item>
                        <el-form-item label="出入类型" prop="type">
                            <el-radio-group v-model="form.type">
                                <el-radio :label=1><span style="font-weight: normal">出校</span></el-radio>
                                <el-radio :label=0><span style="font-weight: normal">入校</span></el-radio>
                            </el-radio-group>
                        </el-form-item>
                        <el-form-item label="身份证号" autocomplete="off"
                                      prop="card">
                            <el-input v-model="form.card" placeholder="请输入身份证号" />
                        </el-form-item>
                        <el-form-item label="所属部门" autocomplete="off"
                                      prop="dept">
                            <el-input v-model="form.dept" placeholder="请输入所属部门" />
                        </el-form-item>
                        <el-form-item label="备注">
                            <el-input v-model="form.remark" type="textarea" placeholder="请输入备注内容"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="submitForm">提交</el-button>
                            <el-button @click="resetForm">重置</el-button>
                        </el-form-item>
                    </el-form>
                </el-card>
            </el-col>
            <el-col :span="17">
                <el-card shadow="hover">
                    <el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
                        <el-form-item label="人员姓名" prop="accept">
                            <el-input
                                    v-model="queryParams.name"
                                    placeholder="请输入人员姓名"
                                    clearable
                                    auto-complete="off"
                                    size="small"
                                    style="width: 138px;"
                                    @keyup.enter.native="handleQuery"
                            />
                        </el-form-item>
                        <el-form-item label="操作类型" prop="type">
                            <el-select
                                    v-model="queryParams.type"
                                    placeholder="请选择出入类型"
                                    clearable
                                    size="small"
                                    style="width: 140px"
                            >
                                <el-option
                                        :key="1"
                                        label="出校"
                                        :value="1"
                                />
                                <el-option
                                        :key="0"
                                        label="入校"
                                        :value="0"
                                />
                            </el-select>
                        </el-form-item>
                        <el-form-item label="出入时间" prop="dateRange">
                            <el-date-picker
                                    v-model="dateRange"
                                    size="small"
                                    style="width: 180px"
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
                    <el-table v-loading="loading" :data="dataList">
                        <el-table-column label="人员姓名" align="center" prop="name" />
                        <el-table-column label="手机号码" width="120" align="center" prop="phone" />
                        <el-table-column prop="type" label="出入类型" width="80" align="center">
                            <template slot-scope="scope">
                                <el-tag size="small" v-if="scope.row.type === 1" type="warning">出校</el-tag>
                                <el-tag size="small" v-else-if="scope.row.type === 0" type="success">入校</el-tag>
                            </template>
                        </el-table-column>
                        <el-table-column label="身份证号" width="168" align="center" prop="card" />
                        <el-table-column label="部门" align="center" prop="dept" />
                        <el-table-column label="操作人" align="center" prop="createBy" />
                        <el-table-column label="时间" align="center" prop="createTime" width="190">
                            <template slot-scope="scope">
                                <span v-if="scope.row.createTime">{{ scope.row.createTime }}</span>
                                <span v-else>无</span>
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
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    export default {
        name: "Register",
        data() {
            const checkCard = (rule, value, callback) => {
                const reg = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/
                if (!value) {
                    return callback(new Error('证件号码不能为空'))
                } else if (!reg.test(value)) {
                    return callback(new Error('证件号码不正确'))
                } else {
                    callback()
                }
            };
            return {
                form: {
                    name: undefined,
                    phone: undefined,
                    type: 1,
                    card: undefined,
                    dept: undefined,
                    remark: undefined
                },
                dataList: [],
                loading: false,
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    type: undefined
                },
                dateRange: [],
                total: 0,
                rules: {
                    name: [
                        { required: true, message: "姓名不能为空", trigger: "blur" }
                    ],
                    type: [
                        { required: true, message: "类型不能为空", trigger: "blur" }
                    ],
                    dept: [
                        { required: true, message: "部门不能为空", trigger: "blur" }
                    ],
                    phone: [
                        { required: true, message: "手机号不能为空", trigger: "blur" },
                        {
                            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
                            message: "请输入正确的手机号码",
                            trigger: "blur"
                        }
                    ],
                    card: [
                        { required: true, validator: checkCard, trigger: 'blur' }
                    ]
                },
            }
        },
        methods: {
            submitForm(){
                this.$refs['form'].validate((valid) => {
                  if (valid){
                      this.$axios.post("/access/register", this.form).then(res => {
                          this.modal.notifySuccess(res.data.data)
                          this.resetForm()
                          this.getList()
                      })
                  }else {
                      this.modal.msgError("填写数据格式错误！")
                      return false
                  }
                })
            },
            resetForm(){
                this.$refs['form'].resetFields()
                this.form = {
                    name: undefined,
                    phone: undefined,
                    type: 1,
                    card: undefined,
                    dept: undefined,
                    remark: undefined
                }
            },
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    type: undefined
                }
                this.dateRange = []
                this.handleQuery()
            },
            handleQuery(){
                this.queryParams.pageNum = 1;
                this.getList()
            },
            getList(){
                this.loading = true
                this.$axios.get("/access/register/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        start: this.dateRange[0] ? this.dateRange[0] : undefined,
                        end: this.dateRange[1] ? this.dateRange[1] :undefined,
                        name: this.queryParams.name,
                        type: this.queryParams.type
                    }}).then(res => {
                    this.dataList = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            }
        },
        created() {
            this.getList()
        }
    }
</script>

<style scoped>

</style>
