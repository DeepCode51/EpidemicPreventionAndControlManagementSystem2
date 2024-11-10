<template>
	<div class="main-class">
		<el-row :gutter="20">
			<el-col :span="8" :xs="24">
				<el-card shadow="hover">
					<div style="text-align: center">
						<UserAvatar :user="user"></UserAvatar>
					</div>
					<ul class="list">
						<li class="list-group-item" style="border-top: 1px solid #e7eaec;">
							<i class="el-icon-user-solid icon" />用户昵称
							<div class="pull-right">{{ user.nickname }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-phone icon" />手机号码
							<div class="pull-right">{{ user.phoneNumber }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-s-home icon" />城市
							<div class="pull-right">{{ user.city }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-time icon" />创建时间
							<div class="pull-right">{{ user.createTime }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-time icon" />修改时间
							<div class="pull-right">{{ user.updateTime }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-s-cooperation icon" />归属部门
							<div class="pull-right">{{ dept.deptName }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-phone-outline icon" />部门电话
							<div class="pull-right">{{ dept.phone }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-more icon" />部门邮箱
							<div class="pull-right">{{ dept.email }}</div>
						</li>
						<li class="list-group-item">
							<i class="el-icon-s-opportunity icon" />备注信息
							<div class="pull-right">{{ user.remark }}</div>
						</li>
					</ul>
				</el-card>
			</el-col>
			<el-col :span="16" :xs="24">
				<el-card shadow="hover">
					<el-tabs v-model="activeName">
						<el-tab-pane label="基本资料" name="basic">
							<el-form ref="basic" :model="basic" :rules="rules" label-width="80px">
								<el-form-item label="用户昵称" prop="nickname">
									<el-input v-model="basic.nickname" maxlength="30" />
								</el-form-item>
								<el-form-item label="手机号码" prop="phoneNumber">
									<el-input v-model="basic.phoneNumber" maxlength="11" />
								</el-form-item>
								<el-form-item label="城市" prop="city">
									<el-input v-model="basic.city" maxlength="11" />
								</el-form-item>
								<el-form-item label="备注信息" prop="remark">
									<el-input v-model="basic.remark" maxlength="11" />
								</el-form-item>
								<el-form-item>
									<el-button type="primary" size="mini" @click="submit">提交保存</el-button>
								</el-form-item>
							</el-form>
						</el-tab-pane>
						<el-tab-pane label="修改密码" name="reset">
							<el-form ref="pwd" :model="pwd" :rules="pwdRules" label-width="80px">
								<el-form-item label="旧密码" prop="oldPassword">
									<el-input v-model="pwd.oldPassword" placeholder="请输入旧密码" type="password" show-password/>
								</el-form-item>
								<el-form-item label="新密码" prop="newPassword">
									<el-input v-model="pwd.newPassword" placeholder="请输入新密码" type="password" show-password/>
								</el-form-item>
								<el-form-item label="确认密码" prop="confirmPassword">
									<el-input v-model="pwd.confirmPassword" placeholder="请确认密码" type="password" show-password/>
								</el-form-item>
								<el-form-item>
									<el-button type="primary" size="mini" @click="reset">保存</el-button>
								</el-form-item>
							</el-form>
						</el-tab-pane>
					</el-tabs>
				</el-card>
			</el-col>
		</el-row>
	</div>
</template>

<script>

	import UserAvatar from "../components/UserAvatar";
	export default {
		name: "UserCenter",
		components: {UserAvatar},
		data() {
			const equalToPassword = (rule, value, callback) => {
				if (this.pwd.newPassword !== value) {
					callback(new Error("两次输入的密码不一致"));
				} else {
					callback();
				}
			};
			return {
				user: {},
				dept: {},
				basic: {
					id: '',
					nickname: '',
					phoneNumber: '',
					city: '',
					remark: ''
				},
				pwd: {
					oldPassword: undefined,
					newPassword: undefined,
					confirmPassword: undefined
				},
				activeName: 'basic',
				// 表单校验
				rules: {
					nickname: [
						{ required: true, message: "用户昵称不能为空", trigger: "blur" }
					],
					phoneNumber: [
						{ required: true, message: "手机号码不能为空", trigger: "blur" },
						{
							pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
							message: "请输入正确的手机号码",
							trigger: "blur"
						}
					]
				},
				pwdRules: {
					oldPassword: [
						{ required: true, message: "旧密码不能为空", trigger: "blur" }
					],
					newPassword: [
						{ required: true, message: "新密码不能为空", trigger: "blur" },
						{ min: 6, max: 20, message: "长度在 6 到 20 个字符", trigger: "blur" }
					],
					confirmPassword: [
						{ required: true, message: "确认密码不能为空", trigger: "blur" },
						{ required: true, validator: equalToPassword, trigger: "blur" }
					]
				}
			}
		},
		created() {
			this.getUserInfo()
		},
		methods: {
			getUserInfo() {
				this.$axios.get("/userInfo").then(res => {
					this.user = res.data.data.user
					this.dept = res.data.data.dept
					this.basic.id = this.user.id
					this.basic.nickname = this.user.nickname
					this.basic.phoneNumber = this.user.phoneNumber
					this.basic.city = this.user.city
					this.basic.remark = this.user.remark
				})
			},
			submit(){
				this.$refs['basic'].validate((valid) => {
					if (valid){
						this.$axios.post("/sys/user/updateInfo", this.basic).then(res => {
							if (res.data.code === 200){
								this.modal.notifySuccess(res.data.data)
							}
							this.getUserInfo()
						})
					}
				})
			},
			reset(){
				this.$refs["pwd"].validate(valid => {
					if (valid) {
						this.$axios.get("/sys/user/updatePassword", {params: {
								oldPassword: this.pwd.oldPassword,
								newPassword: this.pwd.newPassword,
								confirmPassword: this.pwd.confirmPassword
							}}).then(res => {
								if (res.data.code === 200){
									this.modal.notifySuccess(res.data.data)
								}
						})
					}
				});
			}
		}
	}
</script>

<style scoped>
	.pull-right{
		float: right;
		margin-right: 30px;
	}
	.list{
		list-style: none;
	}
	.icon{
		margin-left: 30px;
		margin-right: 15px;
	}
	.list-group-item {
		border-bottom: 1px solid #e7eaec;
		padding: 11px 0;
	}
	ul{
		padding-inline-start: 0;
	}
</style>
