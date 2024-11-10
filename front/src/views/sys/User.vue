<template>
    <div class="main-class">
        <el-row :gutter="20">
            <!--用户数据-->
            <el-col :span="24" :xs="24">
                <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
                    <el-form-item label="用户名称" prop="userName">
                        <el-input
                                v-model="queryParams.userName"
                                placeholder="请输入用户名称"
                                clearable
                                size="small"
                                style="width: 180px"
                                @keyup.enter.native="handleQuery"
                        />
                    </el-form-item>
                    <el-form-item label="手机号码" prop="phoneNumber">
                        <el-input
                                v-model="queryParams.phoneNumber"
                                placeholder="请输入手机号码"
                                clearable
                                size="small"
                                style="width: 180px"
                                @keyup.enter.native="handleQuery"
                        />
                    </el-form-item>
                    <el-form-item label="状态" prop="status">
                        <el-select
                                v-model="queryParams.status"
                                placeholder="角色状态"
                                clearable
                                size="small"
                                style="width: 150px"
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
                <!--按钮-->
                <el-button
                        type="primary"
                        plain
                        icon="el-icon-plus"
                        size="mini"
                        @click="handleAdd"
                        v-permission="['sys:user:save']"
                >新增</el-button>
                <el-button
                        type="danger"
                        plain
                        icon="el-icon-delete"
                        size="mini"
                        :disabled="multiple"
                        @click="handleDelete"
                        v-permission="['sys:user:delete']"
                >删除</el-button>
                <!--表格-->
                <el-table v-loading="loading" :data="userList" @selection-change="handleSelectionChange">
                    <el-table-column type="selection" width="50" align="center" />
                    <el-table-column label="用户编号" align="center" key="id" prop="id" />
                    <el-table-column label="用户名称" align="center" key="username" prop="username" />
                    <el-table-column label="用户昵称" align="center" key="nickname" prop="nickname" />
                    <el-table-column label="手机号码" align="center" key="phoneNumber" prop="phoneNumber" />
                    <el-table-column label="居住地" align="center" key="city" prop="city" />
                    <el-table-column label="状态" align="center" key="status">
                        <template slot-scope="scope">
                            <el-switch
                                    v-model="scope.row.status"
                                    :active-value="1"
                                    :inactive-value="0"
                                    @change="handleStatusChange(scope.row)"
                            ></el-switch>
                        </template>
                    </el-table-column>
                    <el-table-column label="备注" align="center" width="160" key="remark" prop="remark" />
                    <el-table-column label="创建时间" align="center" prop="createTime" width="160">
                        <template slot-scope="scope">
                            <span v-if="scope.row.createTime">{{ scope.row.createTime }}</span>
                            <span v-else>无</span>
                        </template>
                    </el-table-column>
                    <el-table-column
                            label="操作"
                            align="center"
                            width="250"
                            class-name="small-padding fixed-width"
                    >
                        <template slot-scope="scope">
                            <el-button
                                    size="mini"
                                    type="text"
                                    icon="el-icon-edit"
                                    @click="handleUpdate(JSON.stringify(scope.row))"
                                    v-permission="['sys:user:update']"
                            >修改</el-button>
                            <el-button
                                    size="mini"
                                    type="text"
                                    icon="el-icon-delete"
                                    @click="handleDelete(scope.row)"
                                    v-permission="['sys:user:delete']"
                            >删除</el-button>
                            <el-dropdown size="mini" @command="(command) => handleCommand(command, scope.row)">
                                <span class="el-dropdown-link">
                                  <i class="el-icon-d-arrow-right el-icon--right"></i>更多
                                </span>
                                <el-dropdown-menu slot="dropdown">
                                    <el-dropdown-item icon="el-icon-key" command="handleResetPwd" v-permission="['sys:user:repass']">重置密码</el-dropdown-item>
                                    <el-dropdown-item icon="el-icon-circle-check" command="roleHandle" v-permission="['sys:user:role']">分配角色</el-dropdown-item>
                                </el-dropdown-menu>
                            </el-dropdown>
                        </template>
                    </el-table-column>
                </el-table>
            </el-col>
            <Pagination
                    v-show="total>0"
                    :total="total"
                    :page.sync="queryParams.pageNum"
                    :limit.sync="queryParams.pageSize"
                    @pagination="getUserList"
            />
        </el-row>
        <!-- 添加或修改用户配置对话框 -->
        <el-dialog :title="title" :visible.sync="open" width="600px" append-to-body>
            <el-form ref="form" :model="form" :rules="rules" label-width="80px">
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="用户名称" prop="username">
                            <el-input v-model="form.username" placeholder="请输入用户昵称" maxlength="30" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="归属部门" prop="deptId">
                            <el-select v-model="form.deptId" placeholder="请选择归属部门">
                                <template v-for="item in deptOptions">
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
                </el-row>
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="手机号码" prop="phoneNumber">
                            <el-input v-model="form.phoneNumber" placeholder="请输入手机号码" maxlength="11" />
                        </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="居住地" prop="city">
                            <el-input v-model="form.city" placeholder="请输入居住地" maxlength="50" />
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row>
                    <el-col :span="12">
                        <el-form-item label="状态">
                            <el-radio-group v-model="form.status">
                                <el-radio :label=1><span style="font-weight: normal">正常</span></el-radio>
                                <el-radio :label=0><span style="font-weight: normal">禁用</span></el-radio>
                            </el-radio-group>
                        </el-form-item>
                    </el-col>
                </el-row>
                <el-row>
                    <el-col :span="24">
                        <el-form-item label="备注">
                            <el-input v-model="form.remark" type="textarea" placeholder="请输入内容"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitForm('form')">确 定</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
        <!-- 分配权限对话框 -->
        <el-dialog title="分配角色" :visible.sync="roleOpen" width="600px">

            <el-form :model="roleForm">
                <el-tree
                        class="tree-border"
                        :data="roleTreeData"
                        show-checkbox
                        ref="roleTree"
                        node-key="id"
                        :props="roleProps">
                </el-tree>
            </el-form>

            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="roleSubmit">确 定</el-button>
                <el-button @click="roleOpen=false">取 消</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "User",
        data() {
            return{
                // 部门树选项
                deptOptions: undefined,
                // 表单参数
                form: {},
                roleProps: {
                    children: "children",
                    label: "name"
                },
                // 查询参数
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    userName: undefined,
                    phoneNumber: undefined,
                    status: undefined,
                    deptId: undefined
                },
                showSearch: true,
                // 非多个禁用
                multiple: true,
                // 总条数
                total: 0,
                // 用户表格数据
                userList: null,
                //加载动画
                loading: false,
                roleForm: {
                    roleIds: undefined
                },
                roleTreeData: [],
                // 选中数组
                ids: [],
                title: undefined,
                open: false,
                roleOpen: false,
                userId: undefined,
                // 表单校验
                rules: {
                    username: [
                        { required: true, message: "用户名称不能为空", trigger: "blur" },
                        { min: 2, max: 20, message: '用户名称长度必须介于 2 和 20 之间', trigger: 'blur' }
                    ],
                    phoneNumber: [
                        { required: true, message: "手机号不能为空", trigger: "blur" },
                        {
                            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
                            message: "请输入正确的手机号码",
                            trigger: "blur"
                        }
                    ],
                    deptId: [
                        { required: true, message: "归属部门不能为空", trigger: "blur" }
                    ]
                }
            }
        },
        methods: {
            getDeptList(){
                this.$axios.get("/sys/dept/list/true").then(res => {
                    this.deptOptions = res.data.data[0].children
                })
            },
            getUserList(){
                this.loading = true
                this.$axios.get("/sys/user/list", {params: {
                        userName: this.queryParams.userName,
                        phoneNumber: this.queryParams.phoneNumber,
                        status: this.queryParams.status,
                        deptId: this.queryParams.deptId,
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize
                    }}).then(res => {
                    this.userList = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            },
            /** 搜索按钮操作 */
            handleQuery() {
                this.queryParams.pageNum = 1;
                this.getDeptList()
                this.getUserList()
            },
            /** 重置按钮操作 */
            resetQuery() {
                this.queryParams = {
                    pageNum: 1,
                        pageSize: 10,
                        userName: undefined,
                        phoneNumber: undefined,
                        status: undefined,
                        deptId: undefined,
                        dateRange: undefined
                }
                this.resetForm("queryForm");
                this.handleQuery();
            },
            handleStatusChange(row){
                let text = row.status === 1 ? "启用" : "停用"
                this.modal.confirm('确认要"' + text + '""' + row.username + '"用户吗？').then(function() {
                }).then(() => {
                    this.$axios.put("/sys/user", row).then(res => {
                        if (res.data.code === 200){
                            this.modal.msgSuccess(text + "成功")
                        }else {
                            row.status = row.status === 0 ? 1 : 0
                        }
                        this.getUserList()
                    })
                }).catch(() => {
                    row.status = row.status === 0 ? 1 : 0
                })
            },
            resetForm(formName) {
                if (this.$refs[formName]) {
                    this.$refs[formName].resetFields();
                }
            },
            // 表单重置
            reset() {
                this.form = {
                    id: undefined,
                    deptId: undefined,
                    username: undefined,
                    phoneNumber: undefined,
                    status: 1,
                    remark: undefined,
                    roleIds: []
                }
                this.resetForm("form")
            },
            handleAdd(){
                this.reset()
                this.getDeptList()
                this.open = true
                this.title = "添加用户"
            },
            // 取消按钮
            cancel() {
                this.open = false
                this.reset()
            },
            submitForm(formName){
                this.$refs[formName].validate((valid) => {
                    if (valid){
                        if (this.title === "添加用户"){
                            this.$axios.post("/sys/user", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getUserList()
                            })
                        }else if (this.title === "修改用户"){
                            this.$axios.put("/sys/user", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                }
                                this.getUserList()
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
                this.form = JSON.parse(row)
                this.open = true
                this.title = "修改用户"
            },
            /** 重置密码按钮操作 */
            handleResetPwd(row) {
                this.$prompt('请输入"' + row.username + '"的新密码', "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    closeOnClickModal: false,
                    inputPattern: /^.{5,20}$/,
                    inputErrorMessage: "用户密码长度必须介于 5 和 20 之间",
                }).then(({ value }) => {
                    row.password = value
                    this.$axios.post("/sys/user/reset", row).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess("修改成功，新密码是：" + value)
                        }
                    })
                    this.getUserList()
                }).catch(() => {});
            },
            // 更多操作触发
            handleCommand(command, row) {
                switch (command) {
                    case "handleResetPwd":
                        this.handleResetPwd(row);
                        break;
                    case "roleHandle":
                        this.roleHandle(row)
                        break;
                    default:
                        break;
                }
            },
            handleDelete(row){
                const userIds = row.id || this.ids
                if (userIds.length > 1 && userIds.indexOf(1) !== -1){
                    this.modal.notifyWarning("不可以删除管理员")
                }else{
                    this.modal.confirm('确定要删除用户吗？').then(function () {
                    }).then(()=>{
                        this.$axios.delete("/sys/user?ids=" + userIds).then(res => {
                            if (res.data.code === 200){
                                this.modal.notifySuccess(res.data.data)
                                this.getUserList()
                            }
                        })
                    }).catch(()=>{
                        this.getUserList()
                    })
                }

            },
            // 多选框选中数据
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            },
            // 所有菜单节点数据
            getMenuAllCheckedKeys() {
                // 目前被选中的菜单节点
                let checkedKeys = this.$refs.roleTree.getCheckedKeys();
                // 半选中的菜单节点
                let halfCheckedKeys = this.$refs.roleTree.getHalfCheckedKeys();
                checkedKeys.unshift.apply(checkedKeys, halfCheckedKeys);
                return checkedKeys;
            },
            roleHandle(row){
                this.roleOpen = true
                this.userId = row.id
                this.$axios.get("/sys/role/list").then(res => {
                    this.roleTreeData = res.data.data.records
                })
                this.$axios.get("/sys/user/info/" + row.id).then(res => {
                    let roleIds = []
                    res.data.data.forEach(result => {
                        roleIds.push(result.roleId)
                    })
                    this.$refs.roleTree.setCheckedKeys(roleIds)
                })
            },
            roleSubmit(){
                this.roleForm.roleIds = this.getMenuAllCheckedKeys()
                this.$axios.post("/sys/user/userRole/" + this.userId, this.roleForm.roleIds).then(res => {
                    if (res.data.code === 200){
                        this.modal.notifySuccess(res.data.data)
                    }
                    this.getUserList()
                })
                this.roleOpen = false
            }
        },
        created() {
            this.getDeptList()
            this.getUserList()
        }
    }
</script>

<style lang="scss">
.el-table .el-dropdown-link {
    cursor: pointer;
    color: #409EFF;
    margin-left: 5px;
}

.el-table .el-dropdown, .el-icon-arrow-down {
    font-size: 12px;
}
</style>
