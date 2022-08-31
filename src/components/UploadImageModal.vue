<template>
  <div
    class="modal fade"
    tabindex="-1"
    id="uploadImageModal"
    aria-labelledby="uploadImageModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Image upload</h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label for="formFileSm" class="form-label"
              >Supported jpeg and png images only</label
            >
            <input
              class="form-control form-control-sm"
              id="formFileSm"
              type="file"
              accept="image/png, image/gif, image/jpeg"
              @change="handleFileUpload($event)"
            />
          </div>
          <div v-if="urlImage != ''" class="alert alert-success" role="alert">
            <a :href="urlImage">{{ urlImage }}</a>
          </div>
          <div v-if="error != ''" class="alert alert-danger" role="alert">
            {{ error }}
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-secondary"
            data-bs-dismiss="modal"
          >
            Close
          </button>
          <button
            v-if="!flagProgress"
            class="btn btn-primary"
            v-on:click="Upload()"
          >
            Submit
          </button>
          <button
            v-if="flagProgress"
            class="btn btn-primary"
            type="button"
            disabled
          >
            <span
              class="spinner-grow spinner-grow-sm"
              role="status"
              aria-hidden="true"
            ></span>
            Uploading...
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "UploadImageModal",

  data() {
    return {
      src: "https://art.ruhex.ru/images",
      file: "",
      urlImage: "",
      error: "",
      flagStart: false,
      flagProgress: false,
    };
  },
  methods: {
    handleFileUpload() {
      this.file = event.target.files[0];
    },
    async Upload() {
      this.flagProgress = true;
      let formData = new FormData();
      formData.append("file", this.file);
      const requestOptions = {
        method: "POST",
        body: formData,
      };
      const res = await fetch(this.src, requestOptions);
      this.flagProgress = false;
      if (!res.ok) {
        const json = await res.json();
        this.urlImage = "";
        this.error = json.message;
        return;
      }
      this.error = "";
      this.urlImage = res.headers.get("location");
    },
  },
};
</script>
