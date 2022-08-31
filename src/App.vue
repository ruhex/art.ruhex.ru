<template>
  <upload-image-modal />
  <div class="container" style="padding-top: 40px">
    <div class="card" style="max-width: 800px">
      <div class="card-header text-white">
        <button
          :disabled="countDownValue > 1 ? false : true"
          v-on:click="setLikeImage(fileName)"
          type="button"
          class="btn btn-outline-success position-relative"
        >
          <i class="bi bi-hand-thumbs-up-fill"></i>
          <span
            class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-success"
          >
            {{ likeCount }}
            <span class="visually-hidden">liked count</span>
          </span>
        </button>
        <button
          :disabled="countDownValue > 1 ? false : true"
          v-on:click="deleteImage(fileName)"
          type="button"
          class="btn btn-outline-danger"
          style="margin-left: 20px"
        >
          <i class="bi bi-hand-thumbs-down-fill"></i>
        </button>
        <button
          v-on:click="getTop(10)"
          type="button"
          class="btn btn-outline-info"
          style="margin-left: 20px"
        >
          <i class="bi bi-reception-4"> 10</i>
        </button>
        <button
          v-on:click="CreateNotes"
          class="btn btn-outline-light"
          style="margin-left: 20px"
          data-bs-target="#uploadImageModal"
          data-bs-toggle="modal"
          data-bs-dismiss="modal"
        >
          <i class="bi bi-file-earmark-arrow-up"></i>
        </button>
      </div>
      <a :href="'https://art.ruhex.ru/images/' + fileName"
        ><img :src="profile_pic" class="card-img-top" alt=""
      /></a>
    </div>
  </div>
</template>

<script>
import UploadImageModal from "./components/UploadImageModal.vue";
export default {
  name: "App",
  data() {
    return {
      test: "",
      src: "https://art.ruhex.ru/images",
      profile_pic: "",
      count: 0,
      getCount: 0,
      likeCount: 0,
      answer: {},
      timer: "",
      time2: "",
      fileName: "",
      images: [],
      imagesTop: [],
      currentImagebase64: "",
      countDownValue: 3,
      timer3: "",
    };
  },

  async created() {
    await this.GetNewImage();
    await this.GetImageFromArray();
    this.timer = setInterval(this.GetNewImage, 200);
    this.timer2 = setInterval(this.GetImageFromArray, 3000);
  },
  methods: {
    async getTop(count) {
      const res = await fetch(this.src + "/top/" + count);
      const fileNames = await res.json();
      fileNames.forEach(async (name) => {
        const newImage = await this.getBase64Image(this.src + "/" + name);
        const animeImage = {
          fileName: newImage.fileName,
          likeCount: newImage.likeCount,
          ddata: newImage.base64,
        };
        this.imagesTop.push(animeImage);
      });
    },

    async setLikeImage(fileName) {
      this.likeCount++;
      const requestOptions = {
        method: "PUT",
      };
      await fetch(this.src + "/" + fileName, requestOptions);
      await this.GetImageFromArray();
      clearInterval(this.timer2);
      this.timer2 = setInterval(this.GetImageFromArray, 4000);
    },
    async deleteImage(fileName) {
      const requestOptions = {
        method: "DELETE",
      };
      await fetch(this.src + "/" + fileName, requestOptions);
      await this.GetImageFromArray();
      clearInterval(this.timer2);
      this.timer2 = setInterval(this.GetImageFromArray, 4000);
    },

    countDownTimer() {
      this.countDownValue--;
    },

    async getBase64Image(url) {
      const response = await fetch(url);
      const blob = await response.blob();
      const reader = new FileReader();
      await new Promise((resolve, reject) => {
        reader.onload = resolve;
        reader.onerror = reject;
        reader.readAsDataURL(blob);
      });
      return {
        base64: reader.result,
        fileName: response.headers.get("filename"),
        likeCount: response.headers.get("likedcount"),
      };
    },
    async GetNewImage() {
      if (this.images.length < 10) {
        const newImage = await this.getBase64Image(this.src);
        const animeImage = {
          fileName: newImage.fileName,
          likeCount: newImage.likeCount,
          ddata: newImage.base64,
        };
        this.images.push(animeImage);
      }
    },

    async GetImageFromArray() {
      clearInterval(this.timer3);
      this.countDownValue = 4;
      this.timer3 = setInterval(this.countDownTimer, 1000);
      if (this.images.length < 1) {
        await this.GetNewImage();
      }
      const image =
        this.imagesTop.length > 0 ? this.imagesTop.pop() : this.images.pop();
      this.likeCount = image.likeCount;
      this.fileName = image.fileName;
      this.profile_pic = image.ddata;
      this.count++;
      document.title =
        "show: " + this.count + " | preload: " + this.images.length;
    },
  },

  components: {
    UploadImageModal,
  },
};
</script>

<style>
* {
  border-radius: 0 !important;
}

html body {
  background-color: #2c3e50;
  height: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.image {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.card {
  background-color: #1f2c3a34 !important;
  margin: 0 auto;
  float: none;
}

img {
  max-width: 100%;
  height: auto;
  /* border: 1px dashed #970077a2; */
}
</style>
