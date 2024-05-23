<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-r from-blue-400 to-pink-400 font-space-mono">
    <div class="container bg-white bg-opacity-10 backdrop-blur-md rounded-lg p-6 shadow-lg text-white w-full max-w-md">
      <div v-if="!submitted && !error" class="flex flex-col space-y-4">
        <h3>Xcelerate v1.0</h3>
        <form @submit.prevent="submitUrl" class="flex flex-col space-y-4 text-black">
          <input
            type="url"
            v-model="url"
            placeholder="Enter URL here..."
            required
            class="p-2 border border-gray-300 rounded"
          />
          <button
            type="submit"
            class="py-2 px-4 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Submit
          </button>
        </form>
      </div>
      <div v-else-if="error" class="flex flex-col space-y-4">
        <div class="error-message text-red-600">{{ errorMessage }}</div>
        <button
          @click="resetForm"
          class="py-2 px-4 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          Go Back
        </button>
      </div>
      <div v-else class="flex flex-col space-y-4">
        <div class="title text-xl font-semibold">{{ videoData.title }}</div>
        <img :src="videoData.thumbnail" alt="Thumbnail" class="thumbnail rounded w-full mt-4" />
        <select v-model="selectedVersion" class="p-2 border border-gray-300 rounded text-black">
          <option
            v-for="(item, index) in videoData.metadata"
            :key="index"
            :value="index"
          >
            {{ item.format_note }} ({{ item.ext }}, {{ item.resolution || item.ext }})
          </option>
        </select>
        <button
          @click="goToUrl"
          class="py-2 px-4 mt-4 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          Go
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: '',
      submitted: false,
      videoData: null,
      selectedVersion: null,
      error: false,
      errorMessage: '',
    };
  },
  methods: {
    async submitUrl() {
      try {
        const apiUrl = import.meta.env.VITE_API_URL;
        const formData = new URLSearchParams();
        formData.append('url', this.url);

        const response = await fetch(apiUrl, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
          },
          body: formData.toString(),
        });

        const data = await response.json();

        if (data[0].status === 0) {
          this.errorMessage = data[0].message;
          this.error = true;
        } else {
          this.videoData = data[0];
          this.submitted = true;
        }
      } catch (error) {
        console.error('Error:', error);
        this.errorMessage = 'An unexpected error occurred. Please try again later.';
        this.error = true;
      }
    },
    goToUrl() {
      const selectedVersion = this.videoData.metadata[this.selectedVersion];
      window.location.href = selectedVersion.raw_url;
    },
    resetForm() {
      this.url = '';
      this.submitted = false;
      this.error = false;
      this.errorMessage = '';
    },
  },
};
</script>

<style scoped>
.thumbnail {
  max-width: 100%;
  height: auto;
}
</style>