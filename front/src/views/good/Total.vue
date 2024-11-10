<template>
    <div class="main-class">
        <el-row :gutter="20">
            <el-col :span="13">
                <el-card shadow="hover">
                    <Bar v-if="info.total.length > 0 && info.legend.length > 0" :legend="info.legend" :total="info.total" />
                </el-card>
                <el-card shadow="hover" style="margin-top: 20px">
                    <Pie v-if="info.series.length > 0 && info.legend.length > 0" :legend="info.legend" :series="info.series" />
                </el-card>
            </el-col>
            <el-col :span="11">
                <el-card shadow="hover">
                    <!--条件搜索-->
                    <el-form v-permission="['ROLE_service']" :model="queryParams" ref="queryForm" :inline="true">
                        <el-form-item label="物资名称" prop="type">
                            <el-input
                                    v-model="queryParams.name"
                                    placeholder="输入名称"
                                    clearable
                                    size="small"
                                    style="width: 100px;"
                                    @keyup.enter.native="handleQuery"
                            />
                        </el-form-item>
                        <el-form-item label="物资分类" prop="typeId">
                            <el-select v-model="queryParams.typeId" clearable placeholder="请选择类型" size="small" style="width: 120px">
                                <template v-for="child in typeList">
                                    <el-option :label="child.type" :value="child.id">
                                        <span>{{ child.type }}</span>
                                    </el-option>
                                </template>
                            </el-select>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                            <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
                        </el-form-item>
                    </el-form>
                    <!--数据表格-->
                    <el-table v-loading="loading" :data="totalList">
                        <el-table-column label="物资编号" align="center" prop="id" width="80" />
                        <el-table-column label="物资名称" align="center" prop="name" width="120" />
                        <el-table-column label="物资规格" align="center" prop="unit" width="120" />
                        <el-table-column label="物资单位" align="center" prop="size" width="100" />
                        <el-table-column label="物资库存" align="center" prop="total" width="120" />
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
    import Pie from "./Pie";
    import Bar from "./Bar";
    export default {
        name: "Total",
        components: {Pie, Bar},
        data() {
            return {
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    typeId: undefined
                },
                totalList: [],
                loading: false,
                typeList: [],
                total: 0,
                info: {
                    legend: [],
                    total: [],
                    series: []
                }
            }
        },
        methods: {
            resetQuery(){
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    typeId: undefined
                }
                this.resetForm("queryForm")
                this.getList()
            },
            handleQuery(){
                this.queryParams.pageNum = 1
                this.getList()
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields()
                }
            },
            getList(flag){
                this.loading = true
                this.$axios.get("/good/info/total", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        name: this.queryParams.name,
                        typeId: this.queryParams.typeId
                    }}).then(res => {
                    this.total = res.data.data.total
                    this.totalList = res.data.data.records
                    this.loading = false
                    if (flag){
                        this.info.legend = res.data.data.records.map(item => {
                            return item.name
                        })
                        this.info.total = res.data.data.records.map(item => {
                            return item.total
                        })
                        this.info.series = res.data.data.records.map(item => {
                            return {
                                value: item.total,
                                name: item.name
                            }
                        })
                    }
                })
            },
            getTypeList(){
                this.$axios.get("/good/type").then(res => {
                    this.typeList = res.data.data
                })
            },
        },
        created() {
            this.getTypeList()
            this.getList(true)
        }
    }
</script>

<style scoped>

</style>
