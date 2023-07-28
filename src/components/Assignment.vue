<template>
  <div>
    <div class="image-uploader" :class="imagePresent">
      <input type="file" @change="onFileChanged($event)" accept="image/*" capture />
      <span v-if="imagePresent === 'N'"> Click here to upload Image</span>
      <span v-else>Upload Another Image</span>
    </div>
    <div class="image-wrapper">
      <img v-if="previewImage" :src="previewImage" alt="Preview" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      previewImage: null,
      imagePresent: 'N'
    };
  },
  methods: {
    onFileChanged(event) {
      const file = event.target.files[0];
      if (file) {
        this.previewImage = URL.createObjectURL(file);
        this.imagePresent = 'Y';
        console.log(this.previewImage, file);
      }
    },
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
  background-color: white;
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
</style>
