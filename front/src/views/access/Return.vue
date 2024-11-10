<template>
    <div class="main-class">
        <el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
            <el-form-item label="人员姓名" prop="name">
                <el-input
                        v-model="queryParams.name"
                        placeholder="请输入人员姓名"
                        clearable
                        auto-complete="off"
                        size="small"
                        style="width: 160px;"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="归属部门" prop="dept">
                <el-input
                        v-model="queryParams.dept"
                        placeholder="请输入归属部门"
                        clearable
                        auto-complete="off"
                        size="small"
                        style="width: 160px;"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="出校时间" prop="dateRange">
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
            <el-table-column label="人员姓名" align="center" prop="name" />
            <el-table-column label="联系电话" align="center" prop="phone" />
            <el-table-column label="身份证号" align="center" prop="card" />
            <el-table-column label="部门信息" align="center" prop="dept" />
            <el-table-column label="备注信息" align="center" prop="remark" />
            <el-table-column label="出校时间" align="center" prop="createTime" width="190">
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
</template>

<script>
    export default {
        name: "Return",
        data(){
            return{
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    dept: undefined
                },
                dateRange: [],
                loading: false,
                dataList: [],
                total: 0,
            }
        },
        methods: {
            handleQuery(){
                this.queryParams.pageNum = 1;
                this.getList()
            },
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    dept: undefined
                }
                this.dateRange = []
                this.handleQuery()
            },
            getList(){
                this.loading = true
                this.$axios.get("/access/return/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        name: this.queryParams.name,
                        dept: this.queryParams.dept,
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
            this.getList()
        }
    }
</script>

<style scoped>

</style>
