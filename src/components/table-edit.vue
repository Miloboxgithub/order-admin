<template>
	<el-form ref="formRef" :model="form" :rules="rules" :label-width="options.labelWidth">
		<el-row>
			<el-col :span="options.span" v-for="item in options.list">
				<el-form-item :label="item.label" :prop="item.prop">
					<!-- 文本框、数字框、下拉框、日期框、开关、上传 -->
					<el-input v-if="item.type === 'input'" v-model="form[item.prop]" :disabled="item.disabled"
						:placeholder="item.placeholder" clearable></el-input>
					<el-input-number v-else-if="item.type === 'number'" v-model="form[item.prop]"
						:disabled="item.disabled" controls-position="right"></el-input-number>
					<el-select v-else-if="item.type === 'select'" v-model="form[item.prop]" :disabled="item.disabled"
						:placeholder="item.placeholder" clearable>
						<el-option v-for="opt in item.opts" :label="opt.label" :value="opt.value"></el-option>
						<!-- <el-option v-for="timm" :label="item.label" :value="item.value"></el-option> -->
					</el-select>
					<el-date-picker v-else-if="item.type === 'date'" type="date" v-model="form[item.prop]"
						:value-format="item.format"></el-date-picker>
					<el-switch v-else-if="item.type === 'switch'" v-model="form[item.prop]"
						:active-value="item.activeValue" :inactive-value="item.inactiveValue"
						:active-text="item.activeText" :inactive-text="item.inactiveText"></el-switch>
					<el-upload v-else-if="item.type === 'upload'" class="avatar-uploader" action="#"
						:show-file-list="false" :on-success="handleAvatarSuccess">
						<img v-if="form[item.prop]" :src="form[item.prop]" class="avatar" />
						<el-icon v-else class="avatar-uploader-icon">
							<Plus />
						</el-icon>
					</el-upload>
					<slot :name="item.prop" v-else>

					</slot>
				</el-form-item>
			</el-col>
		</el-row>

		<el-form-item>
			<el-button type="primary" @click="saveEdit(formRef)">保 存</el-button>
		</el-form-item>
	</el-form>
</template>

<script lang="ts" setup>
import { FormOption } from '@/types/form-option';
import { FormInstance, FormRules, UploadProps } from 'element-plus';
import { PropType, ref } from 'vue';

const { options, formData, edit, update } = defineProps({
	options: {
		type: Object as PropType<FormOption>,
		required: true
	},
	formData: {
		type: Object,
		required: true
	},
	edit: {
		type: Boolean,
		required: false
	},
	update: {
		type: Function,
		required: true
	}
});
options.list.forEach(item => {
	item.opts=[{
		label: '08:00-09:00',
		value: '08:00-09:00'
	},
	{
		label: '09:00-10:00',
		value: '09:00-10:00'
	},
	{
		label: '10:00-11:00',
		value: '10:00-11:00'
	},
	{
		label: '11:00-12:00',
		value: '11:00-12:00'
	},
	{
		label: '14:00-15:00',
		value: '14:00-15:00'
	},
	{
		label: '15:00-16:00',
		value: '15:00-16:00'
	},
	{
		label: '16:00-17:00',
		value: '16:00-17:00'
	},
	{
		label: '17:00-18:00',
		value: '17:00-18:00'
	},
	{
		label: '18:00-19:00',
		value: '18:00-19:00'
	},
]
})
console.log(edit,'eeeeeeeeeee')
const form = ref({ ...(edit ? formData : {}) });

const rules: FormRules = options.list.map(item => {
	if (item.required) {
		return { [item.prop]: [{ required: true, message: `${item.label}不能为空`, trigger: 'blur' }] };
	}
	return {};
}).reduce((acc, cur) => ({ ...acc, ...cur }), {});


const formRef = ref<FormInstance>();
const saveEdit = (formEl: FormInstance | undefined) => {
	if (!formEl) return;
	formEl.validate(valid => {
		if (!valid) return false;
		update(form.value);
	})
	if(!edit){
	//console.log(form.value,'增添数据');
	createRoom(form.value);
	}
	else{
		//console.log(form.value,'修改数据');
		changeRoom(form.value);
	}
};
const changeRoom = async (e) => {
	try {
        // 定义要发送的数据
        const data = {
			room_id: e.id,
            room_name: e.roomName,
            capacity: e.capacity,
			start_time:e.time.substring(0,5)+':00',
			end_time:e.time.substring(6,11)+':00',
            // ...其他需要的数据字段
        };
		if(data.end_time==null){
			data.end_time = e.endTime
		}
        // 发起 POST 请求
        const response = await fetch('/api/administrator/updmeetingroom', {
            method: 'PUT', 
            headers: {
                'Content-Type': 'application/json', // 设置请求头，告诉服务器发送的是 JSON 数据
                // 根据需要可能还需要添加其他头部信息，如认证令牌等
            },
            body: JSON.stringify(data), // 将 JavaScript 对象转换为 JSON 字符串
        });

        // 检查响应状态
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        // 解析响应数据为 JSON
        const result = await response.json();
        console.log(result); // 输出获取到的数据

        // 处理 result 数据
        // ...

    } catch (error) {
        console.error('There was a problem with the fetch operation:', error);
    }
        }
const createRoom = async (e) => {
    try {
        // 定义要发送的数据
        const data = {
            room_name: e.roomName,
            capacity: parseInt(e.capacity),
			start_time:e.time.substring(0,5)+':00',
			end_time:e.time.substring(6,11)+':00',
            // ...其他需要的数据字段
        };
		if(data.end_time==null){
			data.end_time = e.endTime
		}
        // 发起 POST 请求
        const response = await fetch('/api/administrator/addmeetingroom', {
            method: 'POST', // 指定请求方法为 POST
            headers: {
                'Content-Type': 'application/json', // 设置请求头，告诉服务器发送的是 JSON 数据
                // 根据需要可能还需要添加其他头部信息，如认证令牌等
            },
            body: JSON.stringify(data), // 将 JavaScript 对象转换为 JSON 字符串
        });

        // 检查响应状态
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        // 解析响应数据为 JSON
        const result = await response.json();
        console.log(result); // 输出获取到的数据

        // 处理 result 数据
        // ...

    } catch (error) {
        console.error('There was a problem with the fetch operation:', error);
    }
}
const handleAvatarSuccess: UploadProps['onSuccess'] = (response, uploadFile) => {
	form.value.thumb = URL.createObjectURL(uploadFile.raw!);
};

</script>

<style>
.avatar-uploader .el-upload {
	border: 1px dashed var(--el-border-color);
	border-radius: 6px;
	cursor: pointer;
	position: relative;
	overflow: hidden;
	transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
	border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
	font-size: 28px;
	color: #8c939d;
	width: 178px;
	height: 178px;
	text-align: center;
}
</style>
