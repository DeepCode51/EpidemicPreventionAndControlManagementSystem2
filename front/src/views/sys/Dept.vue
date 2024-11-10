<template>
    <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
    <div class="main-class">
        <!--操作按钮-->
        <el-button
                type="primary"
                plain
                icon="el-icon-plus"
                size="mini"
                @click="handleAdd"
                v-permission="['sys:dept:save']"
        >新增</el-button>
        <el-button
                type="info"
                plain
                icon="el-icon-sort"
                size="mini"
                @click="toggleExpandAll"
        >展开/折叠</el-button>

        <!--数据表格-->
        <el-table
                v-if="refreshTable"
                v-loading="loading"
                :data="deptList"
                row-key="deptId"
                :default-expand-all="isExpandAll"
                :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
        >
            <el-table-column prop="deptName" label="部门名称"></el-table-column>
            <el-table-column prop="orderNum" label="排序" width="80" align="center"></el-table-column>
            <el-table-column prop="leader" label="领导人" width="130" align="center"></el-table-column>
            <el-table-column prop="phone" label="手机号" align="center"></el-table-column>
            <el-table-column prop="email" label="邮件" align="center"></el-table-column>
            <el-table-column prop="status" label="状态" width="80" align="center">
                <template slot-scope="scope">
                    <el-tag size="small" v-if="scope.row.status === 1" type="success">正常</el-tag>
                    <el-tag size="small" v-else-if="scope.row.status === 0" type="danger">禁用</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="创建时间" align="center" prop="createTime">
                <template slot-scope="scope">
                    <span v-if="scope.row.createTime">{{ scope.row.createTime }}</span>
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
                            v-permission="['sys:dept:update']"
                    >修改</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-plus"
                            @click="handleAdd(scope.row)"
                            v-permission="['sys:dept:save']"
                    >新增</el-button>
                    <el-button
                            v-if="scope.row.parentId !== 0"
                            size="mini"
                            type="text"
                            icon="el-icon-delete"
                            @click="handleDelete(scope.row)"
                            v-permission="['sys:dept:delete']"
                    >删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 添加或修改部门对话框 -->
        <el-dialog :title="title" :visible.sync="open" width="600px" append-to-body>
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <el-row>
                    <el-col :span="24" v-if="form.parentId !== 0">
                        <el-form-item label="上级部门" prop="parentId">
                            <el-select v-model="form.parentId" placeholder="请选择归属部门" style="width: 200px">
                                <template v-for="item in deptList">
                                    <el-option :label="item.deptName" :value="item.deptId"></el-option>
                                    <template v-for="child in item.children">
                                        <el-option :label="child.deptName" :value="child.deptId">
                                            <span>{{ "-- " + child.deptName }}</span>
                                        </el-option>
                                        <template v-for="children in child.children">
                                            <el-option :label="children.deptName" :value="children.deptId">
                                                <span>&nbsp;&nbsp;{{ "- " + children.deptName }}</span>
                                            </el-option>
                                        </template>
                                    </template>
                                </template>
                            </el-select>
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="部门名称" prop="deptName">
                            <el-input v-model="form.deptName" placeholder="请输入部门名称" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="显示排序" prop="orderNum">
                            <el-input-number v-model="form.orderNum" controls-position="right" :min="0" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="负责人" prop="leader">
                            <el-input v-model="form.leader" placeholder="请输入负责人" maxlength="20" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="联系电话" prop="phone">
                            <el-input v-model="form.phone" placeholder="请输入联系电话" maxlength="11" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="邮箱" prop="email">
                            <el-input v-model="form.email" placeholder="请输入邮箱" maxlength="50" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="部门状态" prop="status">
                            <el-radio-group v-model="form.status">
                                <el-radio :label=1><span style="font-weight: normal">正常</span></el-radio>
                                <el-radio :label=0><span style="font-weight: normal">禁用</span></el-radio>
                            </el-radio-group>
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
        name: "Dept",
        data() {
            return{
                // 重新渲染表格状态
                refreshTable: true,
                // 遮罩层
                loading: true,
                // 表格树数据
                deptList: undefined,
                // 是否展开，默认全部展开
                isExpandAll: true,
                // 弹出层标题
                title: "",
                // 是否显示弹出层
                open: false,
                // 表单参数
                form: {},
                // 表单校验
                rules: {
                    parentId: [
                        { required: true, message: "上级部门不能为空", trigger: "blur" }
                    ],
                    deptName: [
                        { required: true, message: "部门名称不能为空", trigger: "blur" }
                    ],
                    orderNum: [
                        { required: true, message: "显示排序不能为空", trigger: "blur" }
                    ],
                    email: [
                        {
                            type: "email",
                            message: "'请输入正确的邮箱地址",
                            trigger: ["blur", "change"]
                        }
                    ],
                    phone: [
                        {
                            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
                            message: "请输入正确的手机号码",
                            trigger: "blur"
                        }
                    ]
                }
            }
        },
        methods: {
            getList(){
                this.loading = true
                this.$axios.get("/sys/dept/list/false").then(res => {
                    this.deptList = res.data.data
                    this.loading = false
                })
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields();
                }
            },
            // 表单重置
            reset() {
                this.form = {
                    deptId: undefined,
                    parentId: undefined,
                    deptName: undefined,
                    orderNum: 0,
                    leader: undefined,
                    phone: undefined,
                    email: undefined,
                    status: 1
                };
            },
            // 取消按钮
            cancel() {
                this.open = false;
                this.reset();
                this.getList()
            },
            /** 展开/折叠操作 */
            toggleExpandAll() {
                this.refreshTable = false;
                this.isExpandAll = !this.isExpandAll;
                this.$nextTick(() => {
                    this.refreshTable = true;
                });
            },
            /** 新增按钮操作 */
            handleAdd(row) {
                this.reset();
                if (row !== undefined) {
                    this.form.parentId = row.deptId;
                }
                this.open = true;
                this.title = "添加部门";
            },
            /** 修改按钮操作 */
            handleUpdate(row) {
                this.reset()
                this.form = JSON.parse(row)
                this.open = true
                this.title = "修改部门"
            },
            /** 删除按钮操作 */
            handleDelete(row) {
                this.modal.confirm('是否确认删除名称为"' + row.deptName + '"的部门？').then(function() {
                }).then(() => {
                    this.$axios.delete("/sys/dept/" + row.deptId).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                        }
                    })
                    this.getList();
                }).catch(() => {});
            },
            /** 提交按钮 */
            submitForm: function() {
                this.$refs["form"].validate(valid => {
                    if (valid) {
                        if (this.form.deptId !== undefined) {
                            this.$axios.put("/sys/dept", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getList()
                            })
                        } else {
                            this.$axios.post("/sys/dept", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getList()
                            })
                        }
                    }
                });
                this.open = false
            },
        },
        created() {
            this.getList()
        }
    }
</script>
