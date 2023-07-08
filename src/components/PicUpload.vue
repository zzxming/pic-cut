<script setup lang="ts">
	import 'element-plus/es/components/upload/style/css';
	import 'element-plus/es/components/icon/style/css';
	import { ElUpload, UploadFile, ElIcon } from 'element-plus';
	import { UploadFilled } from '@element-plus/icons-vue';

	const emits = defineEmits(['upload']);

	let upload = (file: UploadFile) => {
		emits('upload', file);
	};
</script>

<template>
	<el-upload
		drag
		accept="image/*"
		:auto-upload="false"
		:show-file-list="false"
		:on-change="upload"
	>
		<el-icon class="el-icon--upload"><upload-filled /></el-icon>
		<div class="el-upload__text">Drop file here or <em>click to upload</em></div>
	</el-upload>
</template>

<style scoped></style>
