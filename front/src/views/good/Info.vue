<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
    <div class="main-class">
        <el-form v-permission="['ROLE_service']" :model="queryParams" ref="queryForm" :inline="true" label-width="68px">
            <el-form-item label="物资名称" prop="name">
                <el-input
                        v-model="queryParams.name"
                        placeholder="请输入物资名称"
                        clearable
                        size="small"
                        style="width: 180px"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="创建人" prop="createBy">
                <el-input
                        v-model="queryParams.createBy"
                        placeholder="请输入创建人"
                        clearable
                        size="small"
                        style="width: 180px"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="状态" prop="status">
                <el-select
                        v-model="queryParams.status"
                        placeholder="物资状态"
                        clearable
                        size="small"
                        style="width: 130px"
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
            <el-form-item label="物资分类" prop="typeId">
                <el-select v-model="queryParams.typeId" clearable placeholder="请选择物资分类" size="small" style="width: 180px">
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

        <!--按钮-->
        <el-button
                type="primary"
                plain
                icon="el-icon-plus"
                size="mini"
                @click="handleAdd"
                v-permission="['good:info:save']"
        >新增</el-button>
        <el-button
                type="danger"
                plain
                icon="el-icon-delete"
                size="mini"
                :disabled="multiple"
                v-permission="['good:info:delete']"
                @click="handleDelete"
        >删除</el-button>

        <!--表格-->
        <el-table v-loading="loading" :data="infoList" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="50" align="center" />
            <el-table-column label="物资编号" align="center" key="id" prop="id" width="80" />
            <el-table-column label="物资名称" align="center" key="name" prop="name" />
            <el-table-column label="物资照片" align="center" prop="img" width="150">
                <template slot-scope="scope">
                    <el-image
                            style="height: 100px"
                            :src="scope.row.img"
                            :preview-src-list="[scope.row.img]">
                    </el-image>
                </template>
            </el-table-column>
            <el-table-column label="物资规格" align="center" key="size" prop="size" width="90" />
            <el-table-column label="物资单位" align="center" key="unit" prop="unit" width="90" />
            <el-table-column label="创建人" align="center" key="createBy" prop="createBy" width="90" />
            <el-table-column label="备注" align="center" key="remark" prop="remark" />
            <el-table-column label="创建时间" align="center" prop="createTime" width="160">
                <template slot-scope="scope">
                    <span v-if="scope.row.createTime">{{ scope.row.createTime }}</span>
                    <span v-else>无</span>
                </template>
            </el-table-column>
            <el-table-column label="更新时间" align="center" prop="updateTime" width="160">
                <template slot-scope="scope">
                    <span v-if="scope.row.updateTime">{{ scope.row.updateTime }}</span>
                    <span v-else>无</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-edit"
                            @click="handleUpdate(JSON.stringify(scope.row))"
                            v-permission="['good:info:update']"
                    >修改</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-delete"
                            @click="handleDelete(scope.row)"
                            v-permission="['good:info:delete']"
                    >删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <Pagination
                v-show="total>0"
                :total="total"
                :page.sync="queryParams.pageNum"
                :limit.sync="queryParams.pageSize"
                @pagination="getInfoList"
        />

        <!-- 添加或修改对话框 -->
        <el-dialog :title="title" :visible.sync="open" width="600px" append-to-body>
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="物资名称" prop="name">
                            <el-input v-model="form.name" placeholder="请输入物资名称" maxlength="30" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="物资规格" prop="size">
                            <el-input v-model="form.size" placeholder="请输入物资规格" maxlength="20" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="物资单位" prop="unit">
                            <el-input v-model="form.unit" placeholder="请输入物资单位" maxlength="20" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="物资分类" prop="typeId">
                            <el-select v-model="form.typeId" placeholder="请选择物资分类">
                                <template v-for="child in typeList">
                                    <el-option :label="child.type" :value="child.id">
                                        <span>{{ child.type }}</span>
                                    </el-option>
                                </template>
                            </el-select>
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
                        <el-form-item label="物资图片" prop="img">
                            <el-upload
                                    class="avatar-uploader"
                                    action="#"
                                    multiple
                                    :show-file-list="false"
                                    :http-request="requestUpload"
                                    :before-upload="beforeAvatarUpload">
                                <img v-if="img" :src="img" class="avatar" alt="">
                                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                            </el-upload>
                        </el-form-item>
                    </el-col>
                    <el-col :span="24">
                        <el-form-item label="备注">
                            <el-input v-model="form.remark" type="textarea" placeholder="请输入备注内容"></el-input>
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
    </el-scrollbar>
</template>

<script>
    export default {
        name: "Info",
        data() {
            return{
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    createBy: undefined,
                    status: undefined,
                    typeId: undefined
                },
                form: {
                    id: undefined,
                    typeId: undefined,
                    img: undefined,
                    name: undefined,
                    size: undefined,
                    unit: undefined,
                    createBy: undefined,
                    status: 1,
                    remark: undefined
                },
                img: undefined,
                srcList: [],
                typeList: [],
                // 非多个禁用
                multiple: true,
                infoList: [],
                //加载动画
                loading: false,
                ids: [],
                file: undefined,
                // 总条数
                total: 0,
                title: undefined,
                open: false,
                // 表单校验
                rules: {
                    name: [
                        { required: true, message: "物资名称不能为空", trigger: "blur" },
                        { min: 2, max: 20, message: '物资名称长度必须介于 2 和 20 之间', trigger: 'blur' }
                    ],
                    typeId: [
                        { required: true, message: "物资分类不能为空", trigger: "blur" }
                    ]
                }
            }
        },
        methods: {
            /** 搜索按钮操作 */
            handleQuery() {
                this.queryParams.pageNum = 1;
                this.getInfoList()
            },
            getTypeList(){
              this.$axios.get("/good/type").then(res => {
                  this.typeList = res.data.data
              })
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields();
                }
            },
            /** 重置按钮操作 */
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    name: undefined,
                    createBy: undefined,
                    status: undefined,
                    typeId: undefined
                }
                this.resetForm("queryForm");
                this.handleQuery();
            },
            // 多选框选中数据
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            },
            getInfoList(){
                this.loading = true
                this.$axios.get("/good/info/list", {params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        name: this.queryParams.name,
                        createBy: this.queryParams.createBy,
                        status: this.queryParams.status,
                        typeId: this.queryParams.typeId
                    }}).then(res => {
                        this.infoList = res.data.data.records
                        this.total = res.data.data.total
                        this.loading = false
                })
            },
             reset() {
                this.form = {
                    id: undefined,
                    typeId: undefined,
                    name: undefined,
                    img: undefined,
                    size: undefined,
                    unit: undefined,
                    status: 1,
                    remark: undefined
                }
                this.file = undefined
                this.img = undefined
            },
            handleAdd(){
                this.reset()
                this.open = true
                this.title = "添加物资"
            },
            cancel() {
                this.open = false
                this.reset()
            },
            // 覆盖默认的上传行为
            requestUpload() {
            },
            beforeAvatarUpload(file) {
                if (file.type.indexOf("image/") === -1) {
                    this.modal.msgError("文件格式错误，请上传图片类型,如：JPG，PNG后缀的文件。")
                } else {
                    const reader = new FileReader()
                    reader.readAsDataURL(file)
                    reader.onload = () => {
                        this.img = reader.result
                    }
                    this.file = file
                }
            },
            handleUpdate(row){
                this.reset()
                this.form = JSON.parse(row)
                this.img = this.form.img
                this.open = true
                this.title = "修改物资"
            },
            handleDelete(row){
                const ids = row.id || this.ids
                this.modal.confirm('确定要删除物资吗？').then(function () {
                }).then(()=>{
                    this.$axios.delete("/good/info?ids=" + ids).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                        this.getInfoList()
                    })
                }).catch(()=>{
                    this.getInfoList()
                })
            },
            submitForm(){
                this.$refs['form'].validate((valid) => {
                    if (valid){
                        if (this.title === '添加物资'){
                            let formData = new FormData();
                            if (this.file !== undefined){
                                formData.append("img", this.file)
                            }
                            formData.append("goodInfo", JSON.stringify(this.form))
                            this.$axios.post("/good/info", formData).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getInfoList()
                            })
                        }else if (this.title === '修改物资') {
                            let formData = new FormData();
                            formData.append("img", this.file)
                            formData.append("goodInfo", JSON.stringify(this.form))
                            this.$axios.put("/good/info", formData).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getInfoList()
                            })
                        }
                    }else{
                        this.modal.msgError('数据格式不正确，请重新输入！')
                        return false
                    }
                    this.open = false
                })
            }
        },
        created() {
            this.getInfoList()
            this.getTypeList()
        }
    }
</script>

<style lang="scss">
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 10px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 110px;
        height: 110px;
        line-height: 110px;
        text-align: center;
    }
    .avatar {
        width: 110px;
        height: 110px;
        display: block;
    }
</style>
