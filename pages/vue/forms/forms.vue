<template>
	<view>
		<text class="example-info">uni-forms 组件一般由输入框、选择器、单选框、多选框等控件组成，用以收集、校验、提交数据。</text>

		<uni-forms  :value="formData" ref="form" validate-trigger="bind" err-show-type="undertext">
			<uni-group title="基本信息" top="0">
				<uni-forms-item name="name" required label="用户名">
					<uni-easyinput type="text" :inputBorder="true" v-model="formData.name" placeholder="请输入用户名"></uni-easyinput>
				</uni-forms-item>
				<!-- 使用原生input，需要绑定binddata -->
				<uni-forms-item name="age" required label="年龄">
					<input type="text" v-model="formData.age" class="uni-input-border" @blur="binddata('age', $event.detail.value)" placeholder="请输入年龄" />
				</uni-forms-item>
				<uni-forms-item name="email" label="邮箱"><uni-easyinput type="text" v-model="formData.email" placeholder="请输入邮箱"></uni-easyinput></uni-forms-item>
				<!-- #ifndef APP-NVUE -->
					<uni-forms-item required name="birth" label="出生日期"><uni-datetime-picker type="date" v-model="formData.birth" start="2000-06-01 06:30:30" end="2030-6-1" return-type="timestamp"></uni-datetime-picker></uni-forms-item>
					<uni-forms-item name="checked" label="详细信息"><switch :checked="formData.checked" @change="change('checked', $event.detail.value)" /></uni-forms-item>
				<!-- #endif -->
			</uni-group>
			<!-- #ifndef APP-NVUE -->
			<template v-if="formData.checked">
				<uni-group title="详细信息">
					<uni-forms-item required name="sex" label="性别"><uni-data-checkbox v-model="formData.sex" :localdata="sex"></uni-data-checkbox></uni-forms-item>
					<uni-forms-item name="country" label="国家">
						<picker :value="formData.country" :range="range" @change="binddata('country', $event.detail.value)">
							<view>{{ formData.country === '' ? '请选择国家' : range[formData.country] }}</view>
						</picker>
					</uni-forms-item>
					<uni-forms-item required name="hobby" label="兴趣爱好"><uni-data-checkbox multiple v-model="formData.hobby" :localdata="hobby" /></uni-forms-item>
					<uni-forms-item name="remarks" label="备注">
						<uni-easyinput type="textarea" v-model="formData.remarks" :maxlength="50" placeholder="请输入备注"></uni-easyinput>
					</uni-forms-item>
				</uni-group>
			</template>
			<!-- #endif -->


			<!-- 直接使用组件自带submit、reset 方法，小程序不生效 -->
			<!-- 			<button class="button" form-type="submit">Submit</button>
				<button class="button" form-type="reset">Reset</button> -->

			<view class="example">
				<button class="button" @click="submitForm('form')">校验表单</button>
				<button class="button" @click="validateField('form')">只校验用户名和邮箱项</button>
				<button class="button" @click="clearValidate('form', 'name')">移除用户名的校验结果</button>
				<button class="button" @click="clearValidate('form')">移除全部表单校验结果</button>
				<button class="button" @click="resetForm">重置表单</button>
			</view>
		</uni-forms>
	</view>
</template>

<script>
export default {
	data() {
		return {
			formData: {
				name: '',
				age: '',
				email: '',
				sex: '',
				hobby: [],
				remarks: '',
				checked: false,
				country: -1,
				birth: ''
			},
			sex: [
				{
					text: '男',
					value: '0'
				},
				{
					text: '女',
					value: '1'
				},
				{
					text: '未知',
					value: '2'
				}
			],
			hobby: [
				{
					text: '足球',
					value: 0
				},
				{
					text: '篮球',
					value: 1
				},
				{
					text: '游泳',
					value: 2
				}
			],
			range: ['中国', '美国', '澳大利亚'],
			show: false,
			rules: {
				name: {
					rules: [
						{
							required: true,
							errorMessage: '请输入用户名'
						},
						{
							minLength: 3,
							maxLength: 15,
							errorMessage: '姓名长度在 {minLength} 到 {maxLength} 个字符'
						}
					]
				},
				age: {
					rules: [
						{
							required: true,
							errorMessage: '请输入年龄'
						},
						{
							format: 'int',
							errorMessage: '年龄必须是数字'
						},
						{
							minimum: 18,
							maximum: 30,
							errorMessage: '年龄应该大于 {minimum} 岁，小于 {maximum} 岁'
						}
					]
				},
				birth: {
					rules: [
						{
							required: true,
							errorMessage: '请选择时间'
						}
					]
				},
				email: {
					rules: [
						{
							format: 'email',
							errorMessage: '请输入正确的邮箱地址'
						}
					]
				},
				checked: {
					rules: [
						{
							format: 'bool'
						}
					]
				},
				sex: {
					rules: [
						{
							format: 'string'
						}
					]
				},
				hobby: {
					rules: [
						{
							format: 'array'
						},
						{
							validateFunction: function(rule, value, data, callback) {
								if (value.length < 2) {
									callback('请至少勾选两个兴趣爱好')
								}
								return true
							}
						}
					]
				}
			}
		}
	},
	onLoad() {
		uni.showLoading()
		// this.formData 应该包含所有需要校验的表单
		// 模拟异步请求数据
		setTimeout(() => {
			this.formData = {
				id:'testId',
				name: 'DCloud',
				age: 21,
				email: '',
				sex: '0',
				hobby: [],
				remarks: '热爱学习，热爱生活',
				checked: false,
				country: 2,
				birth: ''
			}
			uni.hideLoading()
		}, 500)
	},
	onReady() {
		this.$refs.form.setRules(this.rules)
	},
	methods: {
		birthChange(e) {
			console.log(e)
		},
		change(name, value) {
			this.formData.checked = value
			this.$refs.form.setValue(name, value)
		},

		/**
		 * 手动提交
		 * @param {Object} form
		 */
		submitForm(form) {
			this.$refs[form]
				.validate(['id','remarks'])
				.then(res => {
					console.log('表单的值：', res)
					uni.showToast({
						title: '验证成功'
					})
				})
				.catch(errors => {
					console.error('验证失败：', errors)
				})
		},

		/**
		 * 手动重置表单
		 */
		resetForm() {
			this.$refs.form.resetFields()
		},
		/**
		 * 部分表单校验
		 * @param {Object} form
		 */
		validateField(form) {
			this.$refs[form]
				.validateField(['name', 'email'])
				.then(res => {
					uni.showToast({
						title: '验证成功'
					})
					console.log('表单的值：', res)
				})
				.catch(errors => {
					console.error('验证失败：', errors)
				})
		},

		/**
		 * 清除表单校验
		 * @param {Object} form
		 * @param {Object} name
		 */
		clearValidate(form, name) {
			if (!name) name = []
			this.$refs[form].clearValidate(name)
		}
	}
}
</script>

<style lang="scss">
@import '@/common/uni-nvue.scss';

.example {
	padding: 0 10px 10px;
}

.uni-input-border,
.uni-textarea-border {
	// width: 100%;
	flex: 1;
	font-size: 14px;
	color: #666;
	border: 1px #e5e5e5 solid;
	border-radius: 5px;
	/* #ifndef APP-NVUE */
	box-sizing: border-box;
	/* #endif */
}

.uni-input-border {
	padding: 0 10px;
	height: 35px;
}

.uni-textarea-border {
	padding: 10px;
	height: 80px;
}

.label-box {
	margin-right: 10px;
}

.transform-scale {
	transform: scale(0.7);
}

.button {
	margin: 5px 10px;
}
</style>
