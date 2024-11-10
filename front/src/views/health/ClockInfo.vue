<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
    <div class="main-class">
        <el-form v-permission="['ROLE_teacher']" :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
            <el-form-item label="健康状态" prop="healthType">
                <el-select
                        v-model="queryParams.healthType"
                        placeholder="健康状态"
                        clearable
                        size="small"
                        style="width: 130px"
                >
                    <el-option
                            :key="2"
                            label="正常"
                            :value="2"
                    />
                    <el-option
                            :key="1"
                            label="有发热咳嗽"
                            :value="1"
                    />
                    <el-option
                            :key="0"
                            label="其他不正常状况"
                            :value="0"
                    />
                </el-select>
            </el-form-item>
            <el-form-item label="中高风险" prop="middleHigh">
                <el-select
                        v-model="queryParams.middleHigh"
                        placeholder="中高风险"
                        clearable
                        size="small"
                        style="width: 130px"
                >
                    <el-option
                            :key="1"
                            label="否"
                            :value="1"
                    />
                    <el-option
                            :key="0"
                            label="是"
                            :value="0"
                    />
                </el-select>
            </el-form-item>
            <el-form-item label="接触确诊" prop="diagnosis">
            <el-select
                    v-model="queryParams.diagnosis"
                    placeholder="接触确诊"
                    clearable
                    size="small"
                    style="width: 130px"
            >
                <el-option
                        :key="1"
                        label="否"
                        :value="1"
                />
                <el-option
                        :key="0"
                        label="是"
                        :value="0"
                />
            </el-select>
          </el-form-item>
            <el-form-item label="境外返回" prop="returnInfo">
                <el-select
                        v-model="queryParams.returnInfo"
                        placeholder="境外返回"
                        clearable
                        size="small"
                        style="width: 130px"
                >
                    <el-option
                            :key="1"
                            label="否"
                            :value="1"
                    />
                    <el-option
                            :key="0"
                            label="是"
                            :value="0"
                    />
                </el-select>
            </el-form-item>
            <el-form-item label="归属部门" prop="deptId">
                <el-select v-model="queryParams.deptId" placeholder="请选择归属部门" size="small" style="width: 150px">
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
            <el-form-item>
                <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
            </el-form-item>
        </el-form>

        <el-table v-loading="loading" :data="dataList">
            <el-table-column label="编号" align="center" width="80" prop="id" />
            <el-table-column label="学生姓名" align="center" prop="username" />
            <el-table-column prop="healthType" label="健康状况" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.healthType === 2" type="success">正常</el-tag>
                    <el-tag size="small" v-else-if="scope.row.healthType === 1" type="warning">发热咳嗽</el-tag>
                    <el-tag size="small" v-else-if="scope.row.healthType === 0" type="danger">其他不正常</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="returnInfo" label="打卡温度" align="center">
                <template slot-scope="scope">
                    <span>{{ scope.row.temperature }}&nbsp;℃</span>
                </template>
            </el-table-column>
            <el-table-column prop="middleHigh" label="来自中高风险" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.middleHigh === 1" type="success">正常</el-tag>
                    <el-tag size="small" v-else-if="scope.row.middleHigh === 0" type="warning">不正常</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="diagnosis" label="接触确诊" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.diagnosis === 1" type="success">正常</el-tag>
                    <el-tag size="small" v-else-if="scope.row.diagnosis === 0" type="warning">不正常</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="returnInfo" label="接触境外返回" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.returnInfo === 1" type="success">正常</el-tag>
                    <el-tag size="small" v-else-if="scope.row.returnInfo === 0" type="warning">不正常</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="打卡位置" align="center" prop="address" width="190" />
            <el-table-column label="打卡时间" align="center" prop="createTime" width="190">
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
    </div>
    </el-scrollbar>
</template>

<script>
    export default {
        name: "ClockInfo",
        data() {
            return {
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    healthType: undefined,
                    middleHigh: undefined,
                    diagnosis: undefined,
                    returnInfo: undefined,
                    deptId: undefined
                },
                loading: false,
                dataList: [],
                total: 0,
                // 部门树选项
                deptOptions: undefined,
            }
        },
        methods: {
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields();
                }
            },
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    healthType: undefined,
                    middleHigh: undefined,
                    diagnosis: undefined,
                    returnInfo: undefined,
                    deptId: undefined
                }
                this.resetForm("queryForm");
                this.handleQuery();
            },
            handleQuery() {
                this.queryParams.pageNum = 1
                this.getList()
            },
            getList(){
                this.loading = true
                this.$axios.get("/health/clock/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        healthType: this.queryParams.healthType,
                        middleHigh: this.queryParams.middleHigh,
                        diagnosis: this.queryParams.diagnosis,
                        returnInfo: this.queryParams.returnInfo,
                        deptId: this.queryParams.deptId
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
        },
        created() {
            this.getList()
            this.getDeptList()
        }
    }
</script>

<style scoped>

</style>
