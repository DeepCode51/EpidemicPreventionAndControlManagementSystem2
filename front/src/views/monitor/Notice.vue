<template>
    <div class="main-class">
        <el-form :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
            <el-form-item label="公告标题" prop="title">
                <el-input
                        v-model="queryParams.title"
                        placeholder="请输入公告标题"
                        clearable
                        size="small"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="操作人员" prop="createBy">
                <el-input
                        v-model="queryParams.createBy"
                        placeholder="请输入操作人员"
                        clearable
                        size="small"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="公告状态" prop="status">
                <el-select
                        v-model="queryParams.status"
                        placeholder="请选择操作状态"
                        clearable
                        size="small"
                        style="width: 160px"
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

        <!--按钮-->
        <el-button
                type="primary"
                plain
                icon="el-icon-plus"
                size="mini"
                @click="handleAdd"
        >新增</el-button>
        <el-button
                type="danger"
                plain
                icon="el-icon-delete"
                size="mini"
                @click="handleDelete"
                :disabled="multiple"
        >删除</el-button>

        <!--表格-->
        <el-table v-loading="loading" :data="noticeList" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55" align="center" />
            <el-table-column label="序号" align="center" prop="id" width="80" />
            <el-table-column
                    label="公告标题"
                    align="center"
                    prop="title"
                    :show-overflow-tooltip="true"
            />
            <el-table-column label="状态" align="center" key="status" width="100">
                <template slot-scope="scope">
                    <el-switch
                            v-model="scope.row.status"
                            :active-value="1"
                            :inactive-value="0"
                            @change="handleStatusChange(scope.row)"
                    ></el-switch>
                </template>
            </el-table-column>
            <el-table-column label="创建者" align="center" prop="createBy" />
            <el-table-column label="创建时间" align="center" prop="createTime" width="160">
                <template slot-scope="scope">
                    <span v-if="scope.row.createTime">{{ scope.row.createTime }}</span>
                    <span v-else>无</span>
                </template>
            </el-table-column>
            <el-table-column label="更新者" align="center" prop="updateBy">
                <template slot-scope="scope">
                    <span v-if="scope.row.updateBy">{{ scope.row.updateBy }}</span>
                    <span v-else>无</span>
                </template>
            </el-table-column>
            <el-table-column label="更新时间" align="center" prop="updateTime" width="160">
                <template slot-scope="scope">
                    <span v-if="scope.row.updateTime">{{ scope.row.updateTime }}</span>
                    <span v-else>无</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" align="center" class-name="small-padding fixed-width" width="220">
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-edit"
                            @click="handleUpdate(JSON.stringify(scope.row))"
                    >修改</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            @click="handleDelete(scope.row)"
                            icon="el-icon-delete"
                    >删除</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-data-board"
                            @click="submitNotice(scope.row)"
                    >通知</el-button>
                </template>
            </el-table-column>
        </el-table>

        <pagination
                v-show="total>0"
                :total="total"
                :page.sync="queryParams.pageNum"
                :limit.sync="queryParams.pageSize"
                @pagination="getList"
        />
        <!-- 添加或修改公告对话框 -->
        <el-dialog :title="title" :visible.sync="open" width="780px" append-to-body>
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="公告标题" prop="title">
                            <el-input v-model="form.title" placeholder="请输入公告标题" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="状态" prop="status">
                            <el-radio-group v-model="form.status">
                                <el-radio :label=1><span style="font-weight: normal">正常</span></el-radio>
                                <el-radio :label=0><span style="font-weight: normal">禁用</span></el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="内容">
                            <Editor v-model="form.content" :min-height="192"/>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="备注">
                            <el-input v-model="form.remark" type="textarea" placeholder="请输入内容"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitForm">确 定</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    import Editor from "../../components/Editor";
    export default {
        name: "Notice",
        components: {Editor},
        data(){
            return{
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    title: undefined,
                    createBy: undefined,
                    status: undefined
                },
                dateRange: [],
                multiple: true,
                loading: false,
                noticeList: [],
                ids: [],
                total: 0,
                title: undefined,
                open: false,
                form: {
                    id: undefined,
                    title: undefined,
                    content: undefined,
                    status: 1,
                    remark: undefined
                },
                rules: {
                    title: [{ required: true, message: "公告标题不能为空", trigger: "blur" }]
                }
            }
        },
        methods: {
            handleQuery() {
                this.queryParams.pageNum = 1;
                this.getList();
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
                    title: undefined,
                    createBy: undefined,
                    status: undefined
                }
                this.dateRange = []
                this.resetForm("queryForm")
                this.getList()
            },
            getList(){
                this.loading = true
                this.$axios.get("/sys/notice/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        title: this.queryParams.title,
                        createBy: this.queryParams.createBy,
                        status: this.queryParams.status,
                        start: this.dateRange[0],
                        end: this.dateRange[1]
                    }}).then(res => {
                        this.noticeList = res.data.data.records
                        this.total = res.data.data.total
                        this.loading = false
                })
            },
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            },
            handleAdd(){
                this.reset()
                this.open = true
                this.title = "添加公告"
            },
            reset() {
                this.form = {
                    id: undefined,
                    title: undefined,
                    content: undefined,
                    status: 1,
                    remark: undefined
                }
                this.resetForm("form");
            },
            cancel() {
                this.open = false;
                this.reset();
            },
            submitForm(){
                this.$refs['form'].validate((valid) => {
                    if (valid) {
                        if (this.title === "添加公告"){
                            this.$axios.post("/sys/notice", this.form).then(res => {
                                this.modal.notifySuccess(res.data.data)
                            })
                        }else if (this.title === "修改公告"){
                            this.$axios.put("/sys/notice", this.form).then(res => {
                                this.modal.notifySuccess(res.data.data)
                            })
                        }
                        this.getList()
                    }else {
                        this.modal.msgError('数据格式不正确，请重新输入！')
                        return false
                    }
                })
                this.open = false
            },
            handleStatusChange(row){
                let text = row.status === 1 ? "启用" : "停用"
                this.modal.confirm('确认要' + text + '"' + row.title + '"公告吗？').then(function() {
                }).then(() => {
                    this.$axios.put("/sys/notice", row).then(res => {
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
            handleUpdate(row){
                this.reset()
                this.form = JSON.parse(row)
                this.open = true
                this.title = "修改公告"
            },
            handleDelete(row){
                const noticeIds = row.id || this.ids
                this.modal.confirm('确定要删除公告吗？').then(function () {
                }).then(()=>{
                    this.$axios.delete("/sys/notice?ids=" + noticeIds).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                            this.getList()
                        }
                    })
                }).catch(()=>{
                    this.getList()
                })
            },
            submitNotice(row){
                this.modal.confirm('确定要公告这条信息吗？').then(function () {
                }).then(()=>{
                    this.$axios.get("/sys/notice/" + row.id).then(res => {
                        this.modal.notifySuccess(res.data.data)
                    })
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
