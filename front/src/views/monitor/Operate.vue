<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
    <div class="main-class">
        <!--条件查询-->
        <el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
            <el-form-item label="系统模块" prop="title">
                <el-input
                        v-model="queryParams.title"
                        placeholder="请输入系统模块"
                        clearable
                        auto-complete="off"
                        size="small"
                        style="width: 200px;"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="操作人" prop="operName">
                <el-input
                        v-model="queryParams.operName"
                        placeholder="请输入操作人名称"
                        clearable
                        size="small"
                        style="width: 200px;"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="操作状态" prop="status">
                <el-select
                        v-model="queryParams.status"
                        placeholder="请选择操作状态"
                        clearable
                        size="small"
                        style="width: 160px"
                >
                    <el-option
                            :key="1"
                            label="成功"
                            :value="1"
                    />
                    <el-option
                            :key="0"
                            label="失败"
                            :value="0"
                    />
                </el-select>
            </el-form-item>
            <el-form-item label="操作时间" prop="dateRange">
                <el-date-picker
                        v-model="dateRange"
                        size="small"
                        style="width: 240px"
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
        <!--列表-->
        <el-button
                type="danger"
                plain
                icon="el-icon-delete"
                size="mini"
                :disabled="multiple"
                @click="handleDelete"
                v-permission="['monitor:operate:delete']"
        >删除</el-button>
        <el-button
                type="danger"
                plain
                icon="el-icon-delete"
                size="mini"
                @click="handleClean"
                v-permission="['monitor:operate:clear']"
        >清空</el-button>
        <!--数据列表-->
        <el-table ref="tables" v-loading="loading" :data="list" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55" align="center" />
            <el-table-column label="日志编号" align="center" width="80" prop="id" />
            <el-table-column label="系统模块" align="center" prop="title" width="120" />
            <el-table-column label="操作类型" align="center" prop="businessType" width="120" :show-overflow-tooltip="true" />
            <el-table-column label="请求方式" align="center" prop="requestMethod" width="110" :show-overflow-tooltip="true" />
            <el-table-column label="操作人员" align="center" prop="operName" width="120" :show-overflow-tooltip="true" />
            <el-table-column label="操作地址" align="center" prop="operIp" width="130" :show-overflow-tooltip="true" />
            <el-table-column label="操作地点" align="center" prop="operLocation" width="130" :show-overflow-tooltip="true" />
            <el-table-column prop="status" label="状态" width="100" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.status === 1" type="success">成功</el-tag>
                    <el-tag size="small" v-else-if="scope.row.status === 0" type="danger">失败</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="操作日期" align="center" prop="operTime" :show-overflow-tooltip="true" />
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
        <!-- 操作日志详细 -->
        <el-dialog title="操作日志详细" :visible.sync="open" width="700px" append-to-body>
            <el-form ref="form" :model="form" label-width="100px" size="mini">
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="操作模块：">{{ form.title }}</el-form-item>
                        <el-form-item
                                label="登录信息："
                        >{{ form.operName }} / {{ form.operIp }} / {{ form.operLocation }}</el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="请求地址：">{{ form.operUrl }}</el-form-item>
                        <el-form-item label="请求方式：">{{ form.requestMethod }}</el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="操作方法：">{{ form.method }}</el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="请求参数：">{{ form.operParam }}</el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="返回参数：">{{ form.jsonResult }}</el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="操作状态：">
                            <div v-if="form.status === 1">正常</div>
                            <div v-else-if="form.status === 0">失败</div>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="操作时间：">{{ form.operTime }}</el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="异常信息：" v-if="form.status === 0">{{ form.errorMsg }}</el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="open = false">关 闭</el-button>
            </div>
        </el-dialog>
    </div>
    </el-scrollbar>
</template>

<script>
    export default {
        name: "Operate",
        data() {
            return{
                // 查询参数
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    title: undefined,
                    operName: undefined,
                    status: undefined,
                },
                dateRange: [],
                // 非多个禁用
                multiple: true,
                // 选中数组
                ids: [],
                // 表格数据
                list: [],
                // 总条数
                total: 0,
                loading: false,
                open: false,
                // 表单参数
                form: {},
            }
        },
        methods: {
            getList(){
                this.loading = true
                this.$axios.get("/sys/operateLog/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        title: this.queryParams.title,
                        operName: this.queryParams.operName,
                        status: this.queryParams.status,
                        start: this.dateRange[0],
                        end: this.dateRange[1]
                    }}).then(res => {
                    this.list = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields()
                }
            },
            handleQuery(){
                this.queryParams.pageNum = 1;
                this.getList()
            },
            resetQuery() {
                this.resetForm("queryForm")
                this.handleQuery()
            },
            /** 删除按钮操作 */
            handleDelete(row) {
                const infoIds = row.id || this.ids;
                this.modal.confirm('是否确认删除访问编号为"' + infoIds + '"的数据项？').then(function() {
                }).then(() => {
                    this.$axios.delete("/sys/operateLog?ids=" + infoIds).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                        this.getList();
                    })
                }).catch(() => {})
            },
            /** 清空按钮操作 */
            handleClean() {
                this.modal.confirm('是否确认清空所有登录日志数据项？').then(function() {
                }).then(() => {
                    this.$axios.post("/sys/operateLog").then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                        this.getList();
                    })
                }).catch(() => {})
            },
            /** 多选框选中数据 */
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            },
            /** 详细按钮操作 */
            handleView(row) {
                this.open = true
                this.form = row
            },
        },
        created() {
            this.getList()
        }
    }
</script>
