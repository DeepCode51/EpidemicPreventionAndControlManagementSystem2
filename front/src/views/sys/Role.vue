<template>
    <div class="main-class">
        <!--条件搜索-->
        <el-form :model="queryParams" ref="queryForm" :inline="true">
            <el-form-item label="角色名称" prop="roleName">
                <el-input
                        v-model="queryParams.roleName"
                        placeholder="请输入角色名称"
                        clearable
                        size="small"
                        style="width: 210px;"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="权限字符" prop="roleKey">
                <el-input
                        v-model="queryParams.roleKey"
                        placeholder="请输入权限字符"
                        clearable
                        size="small"
                        style="width: 210px"
                        @keyup.enter.native="handleQuery"
                />
            </el-form-item>
            <el-form-item label="状态" prop="status">
                <el-select
                        v-model="queryParams.status"
                        placeholder="角色状态"
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
    <!--按钮-->
        <el-button
                type="primary"
                plain
                icon="el-icon-plus"
                size="mini"
                @click="handleAdd"
                v-permission="['sys:role:save']"
        >新增</el-button>
        <el-button
                type="danger"
                plain
                icon="el-icon-delete"
                size="mini"
                :disabled="multiple"
                @click="handleDelete"
                v-permission="['sys:role:delete']"
        >删除</el-button>
    <!--表格-->
        <el-table v-loading="loading" :data="roleList" @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="50" align="center" />
            <el-table-column label="角色编号" align="center" prop="id" width="160" />
            <el-table-column label="角色名称" prop="name" align="center" :show-overflow-tooltip="true" width="160" />
            <el-table-column label="权限字符" prop="code" align="center" :show-overflow-tooltip="true" width="160" />
            <el-table-column label="状态" align="center">
                <template slot-scope="scope">
                    <el-switch
                            v-model="scope.row.status"
                            :active-value="1"
                            :inactive-value="0"
                            @change="handleStatusChange(scope.row)"
                    ></el-switch>
                </template>
            </el-table-column>
            <el-table-column label="备注" prop="remark" align="center" :show-overflow-tooltip="true" width="180" />
            <el-table-column label="创建时间" align="center" prop="createTime" width="300">
                <template slot-scope="scope">
                    <span>{{ scope.row.createTime }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-edit"
                            @click="handleUpdate(scope.row)"
                            v-permission="['sys:role:update']"
                    >修改</el-button>
                    <el-button
                            size="mini"
                            type="text"
                            icon="el-icon-delete"
                            @click="handleDelete(scope.row)"
                            v-permission="['sys:role:delete']"
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
                <el-form-item prop="name">
                    <span slot="label">
                        <el-tooltip content="该角色的名称，不可重复" placement="top">
                          <i class="el-icon-question"></i>
                        </el-tooltip>
                        角色名称
                      </span>
                    <el-input v-model="form.name" placeholder="请输入角色名称" />
                </el-form-item>
                <el-form-item prop="code">
                      <span slot="label">
                        <el-tooltip content="控制器中定义的权限字符，如：'admin'，不可重复" placement="top">
                          <i class="el-icon-question"></i>
                        </el-tooltip>
                        权限字符
                      </span>
                    <el-input v-model="form.code" placeholder="请输入权限字符" />
                </el-form-item>
                <el-form-item label="状态" prop="status">
                    <el-radio-group v-model="form.status">
                        <el-radio :label=1><span style="font-weight: normal">正常</span></el-radio>
                        <el-radio :label=0><span style="font-weight: normal">禁用</span></el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="菜单权限">
                    <el-checkbox v-model="menuExpand" @change="handleCheckedTreeExpand($event)"><span style="font-weight: normal">展开/折叠</span></el-checkbox>
                    <el-checkbox v-model="menuNodeAll" @change="handleCheckedTreeNodeAll($event, 'menu')"><span style="font-weight: normal">全选/全不选</span></el-checkbox>
                    <el-checkbox v-model="menuCheckStrictly" @change="handleCheckedTreeConnect($event)"><span style="font-weight: normal">父子联动</span></el-checkbox>
                    <el-tree
                            class="tree-border"
                            :data="menuOptions"
                            show-checkbox
                            ref="menu"
                            node-key="id"
                            :check-strictly="!menuCheckStrictly"
                            :props="defaultProps"
                    ></el-tree>
                </el-form-item>
                <el-form-item label="备注">
                    <el-input v-model="form.remark" type="textarea" placeholder="请输入内容"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click="submitForm('form')">确 定</el-button>
                <el-button @click="cancel">取 消</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Role",
        data() {
            return {
                //查询数据
                queryParams: {
                    pageNum: 1,
                    pageSize: 10,
                    roleName: undefined,
                    roleKey: undefined,
                    status: undefined
                },
                // 遮罩层
                loading: false,
                // 角色表格数据
                roleList: [],
                // 总条数
                total: 0,
                // 弹出层标题
                title: "",
                menuCheckStrictly: false,
                // 是否显示弹出层
                open: false,
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                // 菜单列表
                menuOptions: [],
                menuExpand: false,
                menuNodeAll: false,
                // 非多个禁用
                multiple: true,
                // 选中数组
                ids: [],
                // 表单参数
                form: {
                    id: undefined,
                    name: undefined,
                    status: 1,
                    menuIds: [],
                    code: undefined,
                    created: undefined,
                    updated: undefined,
                    remark: undefined
                },
                // 表单校验
                rules: {
                    name: [
                        { required: true, message: "角色名称不能为空", trigger: "blur" }
                    ],
                    code: [
                        { required: true, message: "权限字符不能为空", trigger: "blur" }
                    ]
                }
            }
        },
        methods: {
            getList(){
                this.loading = true
                this.$axios.get("/sys/role/list", {
                    params: {
                        pageNum: this.queryParams.pageNum,
                        pageSize: this.queryParams.pageSize,
                        roleName: this.queryParams.roleName,
                        roleKey: this.queryParams.roleKey,
                        status: this.queryParams.status
                    }
                }).then(res => {
                    this.roleList = res.data.data.records
                    this.total = res.data.data.total
                    this.loading = false
                })
            },
            resetQuery(){
                this.queryParams = {
                    pageNum: 1,
                    pageSize: 10,
                    roleName: undefined,
                    roleKey: undefined,
                    status: undefined
                }
                this.resetForm("queryForm")
                this.getList()
            },
            handleQuery(){
                this.queryParams.pageNum = 1
                this.getList()
            },
            // 角色状态修改
            handleStatusChange(row) {
                let text = row.status === 1 ? "启用" : "停用"
                this.modal.confirm('确认要"' + text + '""' + row.name + '"角色吗？').then(function() {
                }).then(() => {
                    this.$axios.put("/sys/role", row).then(res => {
                        if (res.data.code === 200){
                            this.modal.msgSuccess(text + "成功")
                            this.getList()
                        }else {
                            row.status = row.status === 0 ? 1 : 0
                        }
                    })
                }).catch(() => {
                    row.status = row.status === 0 ? 1 : 0
                })
            },
            reset(){
                if (this.$refs.menu !== undefined) {
                    this.$refs.menu.setCheckedKeys([]);
                }
                this.form = {
                    id: undefined,
                    name: undefined,
                    status: 1,
                    menuIds: [],
                    code: undefined,
                    created: undefined,
                    updated: undefined,
                    remark: undefined
                }
                this.menuExpand = false
                this.menuNodeAll = false
            },
            handleAdd(){
                this.reset()
                this.open = true
                this.title = "添加角色"
                this.$axios.get("/sys/menu/list").then(res => {
                    this.menuOptions = res.data.data
                })
            },
            // 取消按钮
            cancel() {
                this.open = false
                this.reset()
            },
            handleUpdate(row){
                this.reset()
                this.open = true
                this.title = "修改角色"
                this.$axios.get("/sys/role/info/" + row.id).then(res => {
                    this.form = res.data.data
                    this.$axios.get("/sys/menu/list").then(res => {
                        this.menuOptions = res.data.data
                        this.$refs.menu.setCheckedKeys(this.form.menuIds)
                    })
                })
            },
            // 树权限（父子联动）
            handleCheckedTreeConnect(value) {
                this.form.menuCheckStrictly = !!value
            },
            // 树权限（展开/折叠）
            handleCheckedTreeExpand(value) {
                let treeList = this.menuOptions
                for (let i = 0; i < treeList.length; i++) {
                    this.$refs.menu.store.nodesMap[treeList[i].id].expanded = value
                }
            },
            // 树权限（全选/全不选）
            handleCheckedTreeNodeAll(value) {
                this.$refs.menu.setCheckedNodes(value ? this.menuOptions: [])
            },
            // 所有菜单节点数据
            getMenuAllCheckedKeys() {
                // 目前被选中的菜单节点
                let checkedKeys = this.$refs.menu.getCheckedKeys()
                // 半选中的菜单节点
                let halfCheckedKeys = this.$refs.menu.getHalfCheckedKeys()
                checkedKeys.unshift.apply(checkedKeys, halfCheckedKeys)
                return checkedKeys
            },
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid){
                        this.form.menuIds = this.getMenuAllCheckedKeys()
                        this.form.menuCheckStrictly = undefined
                        if (this.title === "添加角色"){
                            this.$axios.post("/sys/role", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                    this.getList()
                                }
                            })
                        }else if (this.title === "修改角色"){
                            this.$axios.put("/sys/role", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                    this.getList()
                                }
                            })
                        }
                    }else{
                        this.modal.msgError('数据格式不正确，请重新输入！')
                        return false
                    }
                })
                this.open = false
            },
            resetForm(formName) {
                if (this.$refs[formName]!==undefined) {
                    this.$refs[formName].resetFields()
                }
            },
            handleDelete(row){
                const roleIds = row.id || this.ids
                if (roleIds.length > 1 && roleIds.indexOf(6) !== -1){
                    this.modal.notifyWarning("不可以删除管理员")
                }else{
                    this.modal.confirm('确定要删除角色吗？').then(function () {
                    }).then(()=>{
                        this.$axios.delete("/sys/role?ids=" + roleIds).then(res => {
                            if (res.data.code === 200){
                                this.modal.notifySuccess(res.data.data)
                                this.getList()
                            }
                        })
                    })

                }
            },
            // 多选框选中数据
            handleSelectionChange(selection) {
                this.ids = selection.map(item => item.id)
                this.multiple = !selection.length
            }
        },
        created() {
            this.getList()
        }
    }
</script>

<style lang="scss">
.tree-border {
    margin-top: 5px;
    border: 1px solid #e5e6e7;
    background: #FFFFFF none;
    border-radius:5px;
}
</style>
