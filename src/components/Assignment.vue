<template>
	<div class="loading-icon" v-if="loading">
		<intersecting-circles-spinner class="apiLoading" :size="80" />
	</div>
	<div>
		<div class="image-uploader" :class="imagePresent">
			<input type="file" @change="onFileChanged($event)" accept="image/*" />
			<span v-if="imagePresent === 'N'"> Click here to upload Image</span>
			<span v-else>Upload Another Image</span>
		</div>
		<div class="image-preview-container">
			<img v-if="previewImage" :src="previewImage" alt="Preview" />
			<section>
				<ul v-if="dominantColor"> Dominant Colors
					<li v-for="color in dominantColor">
						{{ color }}
					</li>
				</ul>
			</section>
			<section>
				<ul v-if="dominantColorScore">
					<li>Color Score: {{ dominantColorScore }}</li>
				</ul>
			</section>
			<section>
				<ul v-if="objects">
					<li>{{ objects }} </li>
				</ul>
			</section>
		</div>
	</div>
</template>

<script>
import axios from 'axios';
import { IntersectingCirclesSpinner } from 'epic-spinners';

export default {
	components: {
		IntersectingCirclesSpinner
	},
	data() {
		return {
			previewImage: null,
			imagePresent: 'N',
			base64Image: '',
			dominantColor: null,
			dominantColorScore: 0,
			objects: null,
			loading: false
		};
	},
	methods: {
		getBase64(file) {
			return new Promise((resolve, reject) => {
				const reader = new FileReader();

				reader.onload = (e) => {
					const base64Image = e.target.result;
					resolve();
					this.base64Image = base64Image.split(',')[1];
				};

				reader.onerror = (e) => {
					reject(e);
				};

				reader.readAsDataURL(file);
			});
		},
		async onFileChanged(event) {
			this.loading = true;
			const file = event.target.files[0];
			if (file) {
				this.previewImage = URL.createObjectURL(file);
				this.imagePresent = 'Y';
				await this.getBase64(file);

				this.getImageDetails(this.base64Image);
			}
		},
		getImageDetails(image) {
			const apiKey = import.meta.env.VITE_API_KEY;

			let data = {
				"requests": [
					{
						"image": {
							"content": image
						},
						"features": [
							{
								"maxResults": 1,
								"type": "IMAGE_PROPERTIES"
							}, {
								"maxResults": 1,
								"type": "OBJECT_LOCALIZATION"
							}
						]
					}
				]
			}

			let config = {
				method: 'post',
				maxBodyLength: Infinity,
				url: 'https://vision.googleapis.com/v1/images:annotate',
				headers: {
					'X-goog-api-key': apiKey,
					'Content-Type': 'application/json; charset=utf-8'
				},
				data: data
			};

			axios.request(config)
				.then((response) => {
					this.loading = false;
					this.dominantColor = Object.keys(response.data.responses[0].imagePropertiesAnnotation.dominantColors.colors[0].color);
					this.dominantColorScore = response.data.responses[0].imagePropertiesAnnotation.dominantColors.colors[0].score;
					try {
						this.objects = "Identified Object: " + response.data.responses[0].localizedObjectAnnotations[0].name;
					} catch (exception) {
						this.objects = "Could not identify object";
					}
				})
				.catch((error) => {
					this.loading = false;
					console.log(error);
				});
		}
	},
};
</script>

<style scoped>
body {
	overflow-y: hidden;
}

.image-uploader {
	align-items: center;
	width: 90vw;
	height: 35vh;
	position: relative;
	display: inline-block;
	border-radius: 8px;
	border: 0.2rem dotted white;
	cursor: pointer;
	text-align: center;
	color: #999;
	font-size: 30px;
	margin-bottom: 0.75rem;
}

.image-uploader:hover {
	background-color: silver;
	color: black;
	border-color: green;
}

.image-uploader input {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	opacity: 0;
}

img {
	display: block;
	width: 50vw;
	height: 50vh;
	border-radius: 8px;
	padding: 10px;
	box-sizing: border-box;
}

.Y {
	border-color: red;
}

.image-preview-container {
	display: flex;
	max-width: 50%;
	align-items: flex-start
}

.image-preview-container section {
	margin-left: 10px;
	font-size: 18px;
	padding-top: 3.5rem;
	padding-left: 3.5rem;
}

.image-preview-container section {
	flex-direction: row;
}

.loading-icon {
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.6);
	display: flex;
	justify-content: center;
	align-items: center;
	z-index: 2;
}</style>
