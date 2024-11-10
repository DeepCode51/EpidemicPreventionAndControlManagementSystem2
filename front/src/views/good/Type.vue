<template>
    <div class="main-class">
        <!--条件搜索-->
        <el-form v-permission="['ROLE_service']" :model="queryParams" ref="queryForm" :inline="true">
            <el-form-item label="物资类型" prop="type">
                <el-input
                        v-model="queryParams.type"
                        placeholder="请输入物资类型"
                        clearable
                        size="small"
                        style="width: 210px;"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="创建人" prop="createBy">
                <el-input
                        v-model="queryParams.createBy"
                        placeholder="请输入创建人"
                        clearable
                        size="small"
                        style="width: 210px"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="状态" prop="status">
                <el-select
                        v-model="queryParams.status"
                        placeholder="类型状态"
                        clearable
                        size="small"
                        style="width: 180px"
                >
                    <el-option
                            :key="1"
                            label="正常"
                            :value="1"
                    />
                    <el-option
                            :key="0"
                            label="禁用"
                            :value="0"
                    />
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
            </el-form-item>
        </el-form>

        <el-button
                type="primary"
                plain
                icon="el-icon-plus"
                size="mini"
                v-permission="['good:type:save']"
                @click="handleAdd"
        >新增</el-button>
        <el-button
                type="danger"
                plain
                icon="el-icon-delete"
                size="mini"
                :disabled="multiple"
                v-permission="['good:type:delete']"
                @click="handleDelete"
        >删除</el-button>

        <!--数据列表-->
        <el-table v-loading="loading" :data="typeList" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="50" align="center" />
            <el-table-column label="类型编号" align="center" prop="id" width="120" />
            <el-table-column label="类型名称" align="center" prop="type" width="150" :show-overflow-tooltip="true" />
            <el-table-column label="排序" align="center" prop="orderNum" width="100" :show-overflow-tooltip="true" />
            <el-table-column label="创建者" align="center" prop="createBy" :show-overflow-tooltip="true" />
            <el-table-column label="状态" align="center" width="100">
                <template slot-scope="scope">
                    <el-switch
                            v-model="scope.row.status"
                            :active-value="1"
                            :inactive-value="0"
                            @change="handleStatusChange(scope.row)"
                    ></el-switch>
                </template>
            </el-table-column>
            <el-table-column label="备注" align="center" prop="remark" :show-overflow-tooltip="true" />
            <el-table-column label="创建时间" align="center" prop="createTime" width="180">
                <template slot-scope="scope">
                    <span>{{ scope.row.createTime }}</span>
                </template>
            </el-table-column>
            <el-table-column label="更新时间" align="center" prop="updateTime" width="180">
                <template slot-scope="scope">
                    <span>{{ scope.row.updateTime }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-edit"
                            v-permission="['good:type:update']"
                            @click="handleUpdate(JSON.stringify(scope.row))"
                    >修改</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-delete"
                            v-permission="['good:type:delete']"
                            @click="handleDelete(scope.row)"
                    >删除</el-button>
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
        <!-- 添加或修改角色配置对话框 -->
        <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
            <el-form ref="form" :model="form" :rules="rules" label-width="100px">
                <el-form-item label="物资分类" prop="type">
                    <el-input v-model="form.type" placeholder="请输入物资分类" />
                </el-form-item>
                <el-form-item label="状态" prop="status">
                    <el-radio-group v-model="form.status">
                        <el-radio :label=1><span style="font-weight: normal">正常</span></el-radio>
                        <el-radio :label=0><span style="font-weight: normal">禁用</span></el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="显示排序" prop="orderNum">
                    <el-input-number v-model="form.orderNum" controls-position="right" :min="1" />
                </el-form-item>
                <el-form-item label="备注" prop="remark">
                    <el-input v-model="form.remark" type="textarea" placeholder="请输入内容"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitForm">确 定</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Type",
        data() {
            return {
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    type: undefined,
                    createBy: undefined,
                    status: undefined
                },
                // 非多个禁用
                multiple: true,
                // 遮罩层
                loading: false,
                typeList: [],
                // 总条数
                total: 0,
                // 弹出层标题
                title: "",
                ids: [],
                // 是否显示弹出层
                open: false,
                form: {
                    id: undefined,
                    type: undefined,
                    status: 1,
                    orderNum: 1,
                    remark: undefined
                },
                // 表单校验
                rules: {
                    type: [
                        { required: true, message: "物资分类不能为空", trigger: "blur" }
                    ],
                    remark: [
                        { required: true, message: "备注不能为空", trigger: "blur" }
                    ]
                }
            }
        },
        methods: {
            getList(){
                this.loading = true
                this.$axios.get("/good/type/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        type: this.queryParams.type,
                        createBy: this.queryParams.createBy,
                        status: this.queryParams.status
                    }}).then(res => {
                        this.typeList = res.data.data.records
                        this.total = res.data.data.total
                        this.loading = false
                })
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields()
                }
            },
            resetQuery(){
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    type: undefined,
                    createBy: undefined,
                    status: undefined
                }
                this.resetForm("queryForm")
                this.getList()
            },
            handleQuery(){
                this.queryParams.pageNum = 1
                this.getList()
            },
            // 多选框选中数据
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            },
            handleStatusChange(row) {
                let text = row.status === 1 ? "启用" : "停用"
                this.modal.confirm('确认要"' + text + '""' + row.type + '"类型吗？').then(function() {
                }).then(() => {
                    this.$axios.put("/good/type", row).then(res => {
                        if (res.data.code === 200){
                            this.modal.msgSuccess(text + "成功")
                        }else {
                            row.status = row.status === 0 ? 1 : 0
                        }
                        this.getList()
                    })
                }).catch(() => {
                    row.status = row.status === 0 ? 1 : 0
                })
            },
            reset(){
                this.form = {
                    id: undefined,
                    type: undefined,
                    status: 1,
                    orderNum: 1,
                    remark: undefined
                }
            },
            handleAdd(){
                this.reset()
                this.open = true
                this.title = "添加类型"
            },
            // 取消按钮
            cancel() {
                this.open = false
                this.reset()
            },
            submitForm() {
                this.$refs.form.validate((valid) => {
                    if (valid){
                        if (this.title === "添加类型"){
                            this.$axios.post("/good/type", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getList()
                            })
                        }else if (this.title === "修改类型"){
                            this.$axios.put("/good/type", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getList()
                            })
                        }
                    }else{
                        this.modal.msgError('数据格式不正确，请重新输入！')
                        return false
                    }
                })
                this.open = false
            },
            handleUpdate(row){
                this.reset()
                this.open = true
                this.title = "修改类型"
                this.form = JSON.parse(row)
            },
            handleDelete(row){
                const infoIds = row.id || this.ids
                this.modal.confirm('是否确认删除访问编号为"' + infoIds + '"的数据项？').then(function() {
                }).then(() => {
                    this.$axios.delete("/good/type?ids=" + infoIds).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                        this.getList();
                    })
                }).catch(() => {})
            }
        },
        created() {
            this.getList()
        }
    }
</script>

<style scoped>

</style>
