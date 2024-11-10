<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
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
            <el-form-item label="返校情况" prop="type">
                <el-select
                        v-model="queryParams.type"
                        placeholder="请输入返校情况"
                        clearable
                        size="small"
                        style="width: 145px"
                >
                    <el-option
                            :key="1"
                            label="假期或请假返校"
                            :value="1"
                    />
                    <el-option
                            :key="0"
                            label="其他"
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
            <el-form-item label="上报时间" prop="dateRange">
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
            <el-form-item>
                <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
            </el-form-item>
        </el-form>

        <!--数据表格-->
        <el-table v-loading="loading" :data="dataList">
            <el-table-column label="编号" align="center" width="80" prop="id" />
            <el-table-column label="学生姓名" align="center" prop="username" width="160" />
            <el-table-column prop="type" label="返校情况" align="center" width="140">
                <template slot-scope="scope">
                    <span v-if="scope.row.type === 1">假期或请假返校</span>
                    <span v-else-if="scope.row.type === 0">其他</span>
                </template>
            </el-table-column>
            <el-table-column label="健康码" align="center" prop="img1" width="150">
                <template slot-scope="scope">
                    <el-image
                            style="height: 100px"
                            :src="scope.row.img1"
                            :preview-src-list="[scope.row.img1]">
                    </el-image>
                </template>
            </el-table-column>
            <el-table-column label="行程码" align="center" prop="img2" width="150">
                <template slot-scope="scope">
                    <el-image
                            style="height: 100px"
                            :src="scope.row.img2"
                            :preview-src-list="[scope.row.img2]">
                    </el-image>
                </template>
            </el-table-column>
            <el-table-column label="核酸报告" align="center" prop="img3">
                <template slot-scope="scope">
                    <el-image
                            style="height: 100px"
                            :src="scope.row.img3"
                            :preview-src-list="[scope.row.img3]">
                    </el-image>
                </template>
            </el-table-column>
            <el-table-column label="手机号码" align="center" prop="phoneNumber" />
            <el-table-column label="提交时间" align="center" prop="createTime" width="190">
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
        name: "ReportInfo",
        data() {
            return{
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    deptId: undefined,
                    username: undefined,
                    type: undefined
                },
                deptOptions: undefined,
                loading: false,
                dataList: [],
                dateRange: [],
                total: 0
            }
        },
        methods: {
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields();
                }
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
                    deptId: undefined,
                    username: undefined,
                    type: undefined
                }
                this.dateRange = []
                this.resetForm("queryForm");
                this.handleQuery();
            },
            handleQuery() {
                this.queryParams.pageNum = 1
                this.getList()
            },
            getList(){
                this.loading = true
                this.$axios.get("/health/report/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        deptId: this.queryParams.deptId,
                        username: this.queryParams.username,
                        type: this.queryParams.type,
                        start: this.dateRange[0] ? this.dateRange[0] : undefined,
                        end: this.dateRange[1] ? this.dateRange[1] :undefined
                    }}).then(res => {
                    this.dataList = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            }
        },
        created() {
            this.getDeptList()
            this.getList()
        }
    }
</script>

<style scoped>

</style>
