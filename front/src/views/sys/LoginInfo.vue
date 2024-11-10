<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
        <div class="main-class">
            <!--条件查询-->
            <el-form :model="queryParams" ref="queryForm" v-show="showSearch" :inline="true" label-width="68px">
                <el-form-item label="登录地址" prop="ip">
                    <el-input
                            v-model="queryParams.ip"
                            placeholder="请输入登录地址"
                            clearable
                            auto-complete="off"
                            size="small"
                            style="width: 200px;"
                            @keyup.enter.native="handleQuery"
                    />
                </el-form-item>
                <el-form-item label="用户名称" prop="username">
                    <el-input
                            v-model="queryParams.username"
                            placeholder="请输入用户名称"
                            clearable
                            size="small"
                            style="width: 200px;"
                            @keyup.enter.native="handleQuery"
                    />
                </el-form-item>
                <el-form-item label="状态" prop="status">
                    <el-select
                            v-model="queryParams.status"
                            placeholder="登录状态"
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
                <el-form-item label="登录时间" prop="dateRange">
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
                    v-permission="['sys:login:delete']"
            >删除</el-button>
            <el-button
                    type="danger"
                    plain
                    icon="el-icon-delete"
                    size="mini"
                    @click="handleClean"
                    v-permission="['sys:login:clear']"
            >清空</el-button>

            <el-table ref="tables" v-loading="loading" :data="list" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55" align="center" />
                <el-table-column label="访问编号" align="center" prop="id" />
                <el-table-column label="用户名称" align="center" prop="username" :show-overflow-tooltip="true" />
                <el-table-column label="登录地址" align="center" prop="ip" width="130" :show-overflow-tooltip="true" />
                <el-table-column label="登录地点" align="center" prop="loginLocation" :show-overflow-tooltip="true" />
                <el-table-column label="浏览器" align="center" prop="browser" :show-overflow-tooltip="true" />
                <el-table-column label="操作系统" align="center" prop="os" />
                <el-table-column prop="status" label="状态" width="100" align="center">
                    <template slot-scope="scope">
                        <el-tag size="small" v-if="scope.row.status === 1" type="success">成功</el-tag>
                        <el-tag size="small" v-else-if="scope.row.status === 0" type="danger">失败</el-tag>
                    </template>
                </el-table-column>
                <el-table-column label="操作信息" align="center" prop="msg" />
                <el-table-column label="登录日期" align="center" prop="loginTime" width="180">
                    <template slot-scope="scope">
                        <span>{{ scope.row.loginTime }}</span>
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
        name: "Dict",
        data() {
            return {
                // 查询参数
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    ip: undefined,
                    username: undefined,
                    status: undefined,
                },
                // 默认排序
                defaultSort: {prop: 'loginTime', order: 'descending'},
                // 非多个禁用
                multiple: true,
                showSearch: true,
                // 选中数组
                ids: [],
                // 表格数据
                list: [],
                // 总条数
                total: 0,
                loading: false,
                dateRange: []
            }
        },
        methods: {
            getList(){
                this.loading = true
                this.$axios.get("/sys/loginInfo/list", {params: {
                        ip: this.queryParams.id,
                        username: this.queryParams.username,
                        status: this.queryParams.status,
                        start: this.dateRange[0],
                        end: this.dateRange[1],
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize
                    }}).then(res => {
                    this.list = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields();
                }
            },
            /** 搜索按钮操作 */
            handleQuery() {
                this.queryParams.pageNum = 1;
                this.getList();
            },
            /** 重置按钮操作 */
            resetQuery() {
                this.resetForm("queryForm")
                this.handleQuery()
            },
            /** 删除按钮操作 */
            handleDelete(row) {
                const infoIds = row.id || this.ids;
                this.modal.confirm('是否确认删除访问编号为"' + infoIds + '"的数据项？').then(function() {
                }).then(() => {
                    this.$axios.delete("/sys/loginInfo?ids=" + infoIds).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                        this.getList();
                    })
                }).catch(() => {});
            },
            /** 清空按钮操作 */
            handleClean() {
                this.modal.confirm('是否确认清空所有登录日志数据项？').then(function() {
                }).then(() => {
                    this.$axios.post("/sys/loginInfo").then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                        this.getList();
                    })
                }).catch(() => {});
            },
            /** 多选框选中数据 */
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            },
        },
        created() {
            this.getList()
        }
    }
</script>

