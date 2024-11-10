<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
    <div class="main-class">
        <el-row :gutter="20">
            <el-col :span="10">
                <el-card shadow="hover">
                    <el-form v-permission="['ROLE_service']" ref="form" :model="form" label-width="80px">
                        <el-form-item label="物资去向" autocomplete="off" :rules="{required: true, message: '物资去向不能为空', trigger: 'blur'}"
                                      prop="accept">
                            <el-input v-model="form.accept" placeholder="请输入物资去向" maxlength="30"/>
                        </el-form-item>
                        <el-form-item label="联系人" autocomplete="off" :rules="{required: true, message: '联系人不能为空', trigger: 'blur'}"
                                      prop="peopleName">
                            <el-input v-model="form.peopleName" placeholder="请输入联系人姓名" />
                        </el-form-item>
                        <el-form-item label="联系电话" autocomplete="off" :rules="rules.phone"
                                      prop="peoplePhone">
                            <el-input v-model="form.peoplePhone" placeholder="请输入联系人电话" />
                        </el-form-item>
                        <el-form-item label="操作类型" prop="status">
                            <el-radio-group v-model="form.operateType">
                                <el-radio :label=1><span style="font-weight: normal">出库</span></el-radio>
                                <el-radio :label=0><span style="font-weight: normal">入库</span></el-radio>
                            </el-radio-group>
                        </el-form-item>
                        <el-form-item label="备注">
                            <el-input v-model="form.remark" type="textarea" placeholder="请输入备注内容"></el-input>
                        </el-form-item>
                        <el-form-item
                                v-for="(domain, index) in form.list"
                                :label="'物资' + (index+1)"
                                :key="domain.key"
                                :prop="'list.' + index + '.goodName'"
                                :rules="rules.good"
                        >
                            <el-select v-model="domain.goodName" placeholder="请选择物资" size="small" style="width: 120px">
                                <template v-for="child in goodList">
                                    <el-option :label="child.name" :value="child.name">
                                        <span>{{ child.name }}</span>
                                    </el-option>
                                </template>
                            </el-select>
                            <el-input-number v-model="domain.goodNum" size="small" style="width: 130px;margin-left: 6px"
                                             :min="1" :max="form.operateType === 1 ? selectNum(domain.goodName) : 100"></el-input-number>
                            <el-tag v-if="domain.goodName" style="margin-left: 6px;">库存：{{ selectNum(domain.goodName, index) }}
                            </el-tag>
                            <el-button size="small" style="margin-left: 6px" @click.prevent="removeDomain(domain)">删除
                            </el-button>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" v-permission="['good:stock:operate']" @click="submitForm">提交</el-button>
                            <el-button v-permission="['good:stock:operate']" @click="addDomain">新增物资</el-button>
                            <el-button v-permission="['good:stock:operate']" @click="resetForm">重置</el-button>
                        </el-form-item>
                    </el-form>
                </el-card>
            </el-col>
            <el-col :span="14">
                <el-card shadow="hover">
                    <el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
                        <el-form-item label="物资去向" prop="accept">
                            <el-input
                                    v-model="queryParams.accept"
                                    placeholder="请输入去向"
                                    clearable
                                    auto-complete="off"
                                    size="small"
                                    style="width: 120px;"
                                    @keyup.enter.native="handleQuery"
                            />
                        </el-form-item>
                        <el-form-item label="操作类型" prop="operateType">
                            <el-select
                                    v-model="queryParams.operateType"
                                    placeholder="请选择类型"
                                    clearable
                                    size="small"
                                    style="width: 120px"
                            >
                                <el-option
                                        :key="1"
                                        label="出库"
                                        :value="1"
                                />
                                <el-option
                                        :key="0"
                                        label="入库"
                                        :value="0"
                                />
                            </el-select>
                        </el-form-item>
                        <el-form-item label="入库时间" prop="dateRange">
                            <el-date-picker
                                    v-model="dateRange"
                                    size="small"
                                    style="width: 215px"
                                    value-format="yyyy-MM-dd"
                                    type="daterange"
                                    range-separator="-"
                                    start-placeholder="开始日期"
                                    end-placeholder="结束日期"
                            ></el-date-picker>
                        </el-form-item>
                        <el-form-item style="float: right;margin-top: -15px">
                            <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                            <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
                        </el-form-item>
                    </el-form>
                    <el-table v-loading="loading" :data="stockList">
                        <el-table-column label="物资去向" align="center" width="130" prop="accept" />
                        <el-table-column label="联系人" align="center" prop="peopleName" />
                        <el-table-column prop="status" label="操作类型" width="100" align="center">
                            <template slot-scope="scope">
                                <el-tag size="small" v-if="scope.row.operateType === 1" type="danger">出库</el-tag>
                                <el-tag size="small" v-else-if="scope.row.operateType === 0" type="success">入库</el-tag>
                            </template>
                        </el-table-column>
                        <el-table-column label="物资名称" align="center" prop="goodName" />
                        <el-table-column label="单位" align="center" prop="goodSize" />
                        <el-table-column label="数量" align="center" prop="goodNum" />
                        <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
                            <template slot-scope="scope">
                                <el-button
                                        size="mini"
                                        type="text"
                                        icon="el-icon-view"
                                        @click="handleView(scope.row)"
                                >详细</el-button>
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
        <el-dialog title="出入库详细信息" :visible.sync="open" width="700px" append-to-body>
            <el-form ref="dataForm" :model="dataForm" label-width="100px" size="mini">
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="操作人员：">{{ dataForm.createBy }}</el-form-item>
                        <el-form-item label="联系人：">{{ dataForm.peopleName }}</el-form-item>
                        <el-form-item label="物资名字：">{{ dataForm.goodName }}</el-form-item>
                        <el-form-item label="物资单位：">{{ dataForm.goodNum }}</el-form-item>
                        <el-form-item label="创建时间：">{{ dataForm.createTime }}</el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="物资去向：">{{ dataForm.accept }}</el-form-item>
                        <el-form-item label="联系电话：">{{ dataForm.peoplePhone }}</el-form-item>
                        <el-form-item label="操作类型：">{{ dataForm.operateType === 0 ? '入库' : '出库' }}</el-form-item>
                        <el-form-item label="物资数量：">{{ dataForm.goodSize }}</el-form-item>
                        <el-form-item label="更新时间：">{{ dataForm.updateTime }}</el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="备注信息：">{{ dataForm.remark }}</el-form-item>
                    </el-col>
                </el-row>
            </el-form>
        </el-dialog>
    </div>
    </el-scrollbar>
</template>

<script>

    export default {
        name: "Stock",
        data() {
            const equalToGoodName = (rule, value, callback) => {
                let resList = this.form.list.map(item => {
                    return item.goodName
                })
                let setLen = new Set(resList).size
                if (resList.length !== setLen){
                    callback(new Error('物资名唯一，不能重复'))
                }
                callback()
            };
            return {
                form: {
                    accept: undefined,
                    peopleName: undefined,
                    peoplePhone: undefined,
                    operateType: 1,
                    remark: undefined,
                    list: [{
                        id: undefined,
                        goodName: undefined,
                        goodNum: 1,
                        goodSize: undefined
                    }]
                },
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    accept: undefined,
                    operateType: undefined
                },
                dateRange: [],
                goodList: [],
                rules: {
                    good: [{
                        required: true, message: '物资不能为空', trigger: 'blur'
                    }, { required: true, validator: equalToGoodName, trigger: 'blur' }],
                    phone: [
                        { required: true, message: "手机号不能为空", trigger: "blur" },
                        {
                            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
                            message: "请输入正确的手机号码",
                            trigger: "blur"
                        }
                    ]
                },
                total: 0,
                loading: false,
                stockList: [],
                open: false,
                // 表单参数
                dataForm: {},
            }
        },
        methods: {
            removeDomain(item) {
                let index = this.form.list.indexOf(item);
                if (index !== -1 && index !== 0) {
                    this.form.list.splice(index, 1)
                }
            },
            addDomain() {
                this.form.list.push({
                    key: Date.now(),
                    goodNum: 1
                })
            },
            submitForm() {
                this.$refs['form'].validate((valid) => {
                    if (valid) {
                        this.$axios.post("/good/stock", this.form).then(res => {
                            this.modal.notifySuccess(res.data.data)
                            this.getList()
                        })
                    } else {
                        return false
                    }
                });
            },
            resetForm() {
                this.$refs['form'].resetFields()
                this.form = {
                    accept: undefined,
                    peopleName: undefined,
                    peoplePhone: undefined,
                    operateType: 1,
                    remark: undefined,
                    list: [{
                        id: undefined,
                        goodName: undefined,
                        goodNum: 1,
                        goodSize: undefined
                    }],
                }
            },
            getGoodList() {
                this.$axios.get("/good/info/all").then(res => {
                    this.goodList = res.data.data
                })
            },
            selectNum(value, index) {
                let total = undefined
                this.goodList.map(item => {
                    if (item.name === value) {
                        total = item.total
                        if (index !== undefined){
                            this.form.list[index].goodSize = item.size
                            this.form.list[index].id = item.id
                        }
                    }
                })
                return total ? total : 0
            },
            getList(){
                this.loading = true
                this.$axios.get("/good/stock/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        accept: this.queryParams.accept,
                        operateType: this.queryParams.operateType,
                        start: this.dateRange[0] ? this.dateRange[0] : undefined,
                        end: this.dateRange[1] ? this.dateRange[1] :undefined
                    }}).then(res => {
                        this.stockList = res.data.data.records
                        this.total = res.data.data.total
                        this.loading = false
                })
            },
            handleQuery(){
                this.queryParams.pageNum = 1;
                this.getList()
            },
            /** 重置按钮操作 */
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    accept: undefined,
                    operateType: undefined
                }
                this.dateRange = []
                this.handleQuery()
            },
            handleView(row) {
                this.open = true
                this.dataForm = row
            },
        },
        created() {
            this.getList()
            this.getGoodList()
        }
    }
</script>

<style scoped>

</style>
