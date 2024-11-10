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
            v-permission="['sys:menu:save']"
    >新增</el-button>
    <el-button
            type="info"
            plain
            icon="el-icon-sort"
            size="mini"
            @click="toggleExpandAll"
    >展开/折叠</el-button>
    <!--表格-->
    <el-table
            v-if="refreshTable"
            v-loading="loading"
            :data="menuList"
            row-key="id"
            :default-expand-all="isExpandAll"
            :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
    >
        <el-table-column prop="name" label="菜单名称" :show-overflow-tooltip="true" width="160"></el-table-column>
        <el-table-column prop="icon" label="图标" align="center" width="100">
            <template slot-scope="scope">
                <i :class="scope.row.icon" />
            </template>
        </el-table-column>
        <el-table-column prop="orderNum" label="排序" width="100" align="center"></el-table-column>
        <el-table-column prop="perms" label="权限标识" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="type" label="类型" align="center" :show-overflow-tooltip="true">
            <template slot-scope="scope">
                <el-tag size="small" v-if="scope.row.type === 0">目录</el-tag>
                <el-tag size="small" v-else-if="scope.row.type === 1" type="warning">菜单</el-tag>
                <el-tag size="small" v-else-if="scope.row.type === 2" type="info">按钮</el-tag>
            </template>
        </el-table-column>
        <el-table-column prop="component" label="组件路径" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="status" label="状态" width="100" align="center">
            <template slot-scope="scope">
                <el-tag size="small" v-if="scope.row.status === 1" type="success">正常</el-tag>
                <el-tag size="small" v-else-if="scope.row.status === 0" type="danger">禁用</el-tag>
            </template>
        </el-table-column>
        <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
            <template slot-scope="scope">
                <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-edit"
                        @click="handleUpdate(scope.row)"
                        v-permission="['sys:menu:update']"
                >修改</el-button>
                <el-button v-if="scope.row.type !== 2"
                        size="mini"
                        type="text"
                        icon="el-icon-plus"
                        @click="handleAdd(scope.row)"
                        v-permission="['sys:menu:save']"
                >新增</el-button>
                <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-delete"
                        @click="handleDelete(scope.row)"
                        v-permission="['sys:menu:delete']"
                >删除</el-button>
            </template>
        </el-table-column>
    </el-table>
    <!-- 添加或修改菜单对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="680px" append-to-body>
        <el-form ref="form" :model="form" :rules="rules" label-width="100px">
            <el-row>
                <el-col :span="24">
                    <el-form-item label="上级菜单" prop="parentId">
                        <el-select v-model="form.parentId" placeholder="请选择上级菜单">
                            <el-option label="无" :value="0"></el-option>
                            <template v-for="item in menuList">
                                <el-option :label="item.name" :value="item.id"></el-option>
                                <template v-for="child in item.children">
                                    <el-option :label="child.name" :value="child.id">
                                        <span>{{ "- " + child.name }}</span>
                                    </el-option>
                                </template>
                            </template>
                        </el-select>
                    </el-form-item>
                </el-col>
                <el-col :span="24">
                    <el-form-item label="类型" prop="type" label-width="100px">
                        <el-radio-group v-model="form.type">
                            <el-radio :label=0><span style="font-weight: normal">目录</span></el-radio>
                            <el-radio :label=1><span style="font-weight: normal">菜单</span></el-radio>
                            <el-radio :label=2><span style="font-weight: normal">按钮</span></el-radio>
                        </el-radio-group>
                    </el-form-item>
                </el-col>
                <el-col :span="24" v-if="form.type !== 2">
                    <el-form-item label="菜单图标">
                        <el-popover
                                placement="bottom-start"
                                width="460"
                                trigger="click"
                        >
                            <div class="icon-body">
                                <div class="icon-list">
                                    <div v-for="(item) in iconList" :key="item" @click="selected(item)">
                                        <span style=""><i :class="item" style="margin-right: 10px" />{{ item.split('el-icon-')[1] }}</span>
                                    </div>
                                </div>
                            </div>
                            <el-input slot="reference" v-model="form.icon" placeholder="点击选择图标" readonly>
                                <i
                                        v-if="form.icon"
                                        slot="prefix"
                                        :class="form.icon"
                                        style="height: 32px;width: 16px;"
                                />
                                <i v-else slot="prefix" class="el-icon-search el-input__icon" />
                            </el-input>
                        </el-popover>
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-form-item label="菜单名称" prop="name">
                        <el-input v-model="form.name" placeholder="请输入菜单名称" />
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-form-item label="显示排序" prop="orderNum">
                        <el-input-number v-model="form.orderNum" controls-position="right" :min="1" />
                    </el-form-item>
                </el-col>
                <el-col :span="12" v-if="form.type === 1">
                    <el-form-item label="路由地址" prop="path">
                        <span slot="label">
                            <el-tooltip content="访问的路由地址，如：`user`" placement="top">
                            <i class="el-icon-question"></i>
                            </el-tooltip>
                            路由地址
                        </span>
                        <el-input v-model="form.path" placeholder="请输入路由地址" />
                    </el-form-item>
                </el-col>
                <el-col :span="12" v-if="form.type ===1">
                    <el-form-item prop="component">
                          <span slot="label">
                            <el-tooltip content="访问的组件路径，如：`sys/user/index`，默认在`views`目录下" placement="top">
                            <i class="el-icon-question"></i>
                            </el-tooltip>
                            组件路径
                          </span>
                        <el-input v-model="form.component" placeholder="请输入组件路径" />
                    </el-form-item>
                </el-col>

                <el-col :span="12">
                    <el-form-item prop="perms">
                        <el-input v-model="form.perms" placeholder="请输入权限标识" maxlength="100" />
                        <span slot="label">
                            <el-tooltip content="控制器中定义的权限字符，如：'system:user:list'" placement="top">
                            <i class="el-icon-question"></i>
                            </el-tooltip>
                            权限字符
                        </span>
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
            </el-row>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button type="primary" @click="submitForm('form')">确 定</el-button>
            <el-button @click="cancel">取 消</el-button>
        </div>
    </el-dialog>
</div>
    </el-scrollbar>
</template>

<script>
    export default {
        name: "Menu",
        data() {
            return{
                // 遮罩层
                loading: false,
                // 重新渲染表格状态
                refreshTable: true,
                // 是否展开，默认全部折叠
                isExpandAll: false,
                // 菜单表格树数据
                menuList: [],
                // 查询参数
                queryParams: {
                    menuName: undefined,
                    visible: undefined
                },
                // 显示搜索条件
                showSearch: true,
                // 是否显示弹出层
                open: false,
                // 弹出层标题
                title: "",
                // 表单参数
                form: {
                    type: 0,
                    icon: '',
                    orderNum: 1,
                    status: 1,
                    parentId: 0
                },
                // 表单校验
                rules: {
                    name: [
                        { required: true, message: "菜单名称不能为空", trigger: "blur" }
                    ],
                    orderNum: [
                        { required: true, message: "菜单顺序不能为空", trigger: "blur" }
                    ],
                    path: [
                        { required: true, message: "路由地址不能为空", trigger: "blur" }
                    ],
                    type: [
                        { required: true, message: "菜单类型不能为空", trigger: "blur" }
                    ],
                    status: [
                        { required: true, message: "状态不能为空", trigger: "blur" }
                    ],
                    perms: [
                        { required: true, message: "权限不能为空", trigger: "blur" }
                    ]
                },
                iconList: [
                    "el-icon-delete",
                    "el-icon-setting",
                    "el-icon-user-solid",
                    "el-icon-phone",
                    "el-icon-more",
                    "el-icon-star-on",
                    "el-icon-goods",
                    "el-icon-picture",
                    "el-icon-upload",
                    "el-icon-message-solid",
                    "el-icon-s-platform",
                    "el-icon-s-open",
                    "el-icon-s-promotion",
                    "el-icon-s-opportunity",
                    "el-icon-s-custom",
                    "el-icon-s-data",
                    "el-icon-cpu",
                    "el-icon-chat-round",
                    "el-icon-chat-square",
                    "el-icon-odometer",
                    "el-icon-switch-button",
                    "el-icon-lock",
                    "el-icon-location",
                    "el-icon-message",
                    "el-icon-set-up",
                    "el-icon-paperclip",
                    "el-icon-loading"
                ]
            }
        },
        methods: {
            getList(){
                this.loading = true
                this.$axios.get("/sys/menu/list").then(res => {
                    this.menuList = res.data.data
                    this.loading = false
                })
            },
            selected(name) {
                this.form.icon = name
                document.body.click()
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
                if (row != null && row.id) {
                    this.form.parentId = row.id
                    this.form.type = row.type + 1
                }
                this.open = true
                this.title = "添加菜单"
            },
            // 取消按钮
            cancel() {
                this.open = false;
                this.reset()
            },
            reset(){
                this.form = {
                    type: 0,
                    icon: '',
                    orderNum: 1,
                    status: 1,
                    parentId: 0
                }
            },
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid){
                        if (this.title === '添加菜单'){
                            this.$axios.post("/sys/menu", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                    this.getList();
                                }
                            })
                        }else if (this.title === '修改菜单'){
                            this.$axios.put("/sys/menu", this.form).then(res => {
                                if (res.data.code === 200){
                                    this.modal.notifySuccess(res.data.data)
                                    this.getList();
                                }
                            })
                        }
                    }else{
                        this.modal.msgError('数据格式不正确，请重新输入！')
                        return false
                    }
                })
                this.open = false
                this.reset()
            },
            handleUpdate(row){
                this.reset()
                this.form = row
                this.open = true
                this.title = "修改菜单"
            },
            handleDelete(row){
                let type = null
                if (row.type === 0){
                    type = '目录'
                }else if (row.type === 1){
                    type = '菜单'
                }else {
                    type = '按钮'
                }
                this.modal.confirm('确定要删除id为 ' + row.id + ' 的' + type + '吗？').then(function() {
                }).then(() => {
                    this.$axios.delete("/sys/menu/" + row.id).then(res => {
                        if (res.data.code === 200){
                            this.modal.notifySuccess(res.data.data)
                            this.getList();
                        }
                    })
                }).catch(function() {
                    this.modal.msgError("删除失败")
                })
            }
        },
        created() {
            this.getList()
        }
    }
</script>

<style lang="scss">
label{
    font-weight: 700 !important;
    color: #606266 !important;
}
.el-table {
    margin-top: 10px;
    border-radius: 5px;
    .el-table__header-wrapper, .el-table__fixed-header-wrapper {
        th {
            word-break: break-word;
            background-color: #f8f8f9;
            color: #515a6e;
            height: 40px;
            font-size: 13px;
        }
    }
    .el-table__body-wrapper {
        .el-button [class*="el-icon-"] + span {
            margin-left: 1px;
        }
    }
}
.icon-body {
    width: 100%;
    padding: 10px;
    .icon-list {
        height: 200px;
        overflow-y: scroll;
        div {
            height: 30px;
            line-height: 30px;
            margin-bottom: -5px;
            cursor: pointer;
            width: 33%;
            float: left;
        }
        span {
            display: inline-block;
            vertical-align: -0.15em;
            fill: currentColor;
            overflow: hidden;
        }
    }
}
</style>
