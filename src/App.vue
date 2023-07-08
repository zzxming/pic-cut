<script setup lang="ts">
	import { ref, computed } from 'vue';
	import PicUpload from './components/PicUpload.vue';
	import CutPic from './components/CutPic.vue';

	import Cropper from 'cropperjs';
	import { UploadFile } from 'element-plus';
	/** 上传图片对象 */
	let picFile = ref<UploadFile>();
	/** 上传图片预览 url */
	let previewUploadFile = computed(() => {
		let file = picFile.value?.raw;
		if (file) {
			return URL.createObjectURL(file);
		}
		return '';
	});
	/** 剪裁弹窗显示 */
	let dialogVisible = ref(false);
	/** 剪裁后图片对象 */
	let cutResult = ref<File>();
	/** 剪裁后图片预览 url */
	let previewCutImage = computed(() => {
		let file = cutResult.value;
		if (file) {
			return URL.createObjectURL(file);
		}
		return '';
	});
	/** 获取上传文件 */
	let getUploadFile = (file: UploadFile) => {
		picFile.value = file;
		dialogVisible.value = true;
	};
	/** 获取剪裁后图片 */
	let getCutPic = (blob: Blob, position: Cropper.Data) => {
		console.log(position);
		console.log(blob);
		cutResult.value = new File([blob], `cut-image.${blob.type.split('/')[1]}`, { type: blob.type });
		dialogVisible.value = false;
	};
	/** 下载剪裁图片 */
	let downCut = () => {
		if (cutResult.value) {
			const link = document.createElement('a');
			const objectUrl = URL.createObjectURL(cutResult.value);

			link.href = objectUrl;
			link.download = cutResult.value.name;
			document.body.appendChild(link);
			link.click();

			document.body.removeChild(link);
			URL.revokeObjectURL(objectUrl);
		}
	};
</script>

<template>
	<div class="wrap">
		<PicUpload
			class="item"
			@upload="getUploadFile"
		/>
		<el-dialog
			v-model="dialogVisible"
			width="80%"
			:destroy-on-close="true"
		>
			<CutPic
				:origin-img="previewUploadFile"
				:img-type="picFile!.raw!.type"
				@cut="getCutPic"
			/>
		</el-dialog>
		<img
			class="item"
			:src="previewCutImage"
		/>
		<el-button
			v-if="cutResult"
			type="primary"
			@click="downCut"
		>
			下载剪裁图片
		</el-button>
	</div>
</template>

<style scoped>
	.wrap {
		display: flex;
		flex-direction: column;
		width: 600px;
	}
	.item {
		width: 100%;
		margin: 0 auto;
	}
</style>
