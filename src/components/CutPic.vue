<script lang="ts" setup>
	import { ref } from 'vue';

	import Cropper from 'cropperjs';
	import 'cropperjs/dist/cropper.min.css';
	import GIF from 'gif.js';
	import { GifToCanvas } from '../assets/js/gifToCanvas';

	const props = defineProps({
		originImg: {
			type: String,
			required: true,
		},
		imgType: {
			type: String,
			required: true,
		},
		aspectRatio: {
			type: Number,
			defualt: 5 / 3,
		},
	});
	const emits = defineEmits(['cut']);

	const cropOption = {
		offset_x: 30,
		offset_y: 40,
		width: 600,
		height: 400,
	};
	const cropImg = ref<HTMLImageElement>();

	let cropper = ref<Cropper>();
	/** 初始化剪裁框 */
	let initCropper = () => {
		if (!cropImg.value) return;

		cropper.value = new Cropper(cropImg.value, {
			checkCrossOrigin: true,
			viewMode: 3,
			zoomOnWheel: false, // 是否可以通过移动鼠标来放大图像
			aspectRatio: props.aspectRatio,
			ready: () => {
				cropper.value?.setData({
					x: cropOption.offset_x,
					y: cropOption.offset_y,
					width: cropOption.width,
					height: cropOption.height,
				});
			},
		});
	};
	/** 获取剪裁后数据加载 */
	let cutLoading = ref(false);
	/** 确认剪裁 */
	let sureCropper = () => {
		const cropParam = cropper.value?.getData();
		cutLoading.value = true;
		// 判断是否 gif
		if (props.imgType.split('/')[1] === 'gif') {
			gifCut(cropParam!);
			return;
		}
		cropper.value?.getCroppedCanvas().toBlob((blob) => {
			emits('cut', blob, cropParam);
			cutLoading.value = false;
		}, 'image/png');
	};
	/** gif 剪裁 */
	let gifCut = (cropParam: Cropper.Data) => {
		const url = (cropper.value as any).url;
		const cropBoxData = cropper.value!.getCropBoxData();
		const canvasData = cropper.value!.getCanvasData();
		const gifToCanvas = new GifToCanvas(url, {
			targetOffset: {
				dx: cropBoxData.left - canvasData.left,
				dy: cropBoxData.top - canvasData.top,
				width: canvasData.width,
				height: canvasData.height,
				sWidth: cropBoxData.width,
				sHeight: cropBoxData.height,
			},
		});
		const gif = new GIF({
			workers: 2,
			quality: 10,
			workerScript: `/static/js/gif.worker.js`,
		});
		const addFrame = (canvas: HTMLCanvasElement, delay: number) => {
			gif.addFrame(canvas, { copy: true, delay });
		};
		gifToCanvas.on('progress', (canvas, delay) => {
			addFrame(canvas, delay);
		});
		gifToCanvas.on('finished', (canvas, delay) => {
			addFrame(canvas, delay);
			gif.render();
		});
		gif.on('finished', (blob) => {
			emits('cut', blob, cropParam);
			cutLoading.value = false;
			return;
		});
		gifToCanvas.init();
	};
</script>

<template>
	<div class="cut">
		<div class="cut_wrapper">
			<img
				ref="cropImg"
				:id="originImg"
				:src="originImg"
				@load="initCropper"
			/>
		</div>
		<el-button
			@click="sureCropper"
			:loading="cutLoading"
		>
			确认剪裁
		</el-button>
	</div>
</template>

<style scoped>
	.cut {
		display: flex;
		flex-direction: column;
	}
	img {
		width: 100%;
		height: 100%;
		object-fit: contain;
		margin: 0 auto;
		display: block;
	}
</style>
