<template>
  <div class="home">
    <div class="header">
      <h1>
        This is just a test
      </h1>
      <div class="header-tools">
        <ThemeToggle />
      </div>
    </div>

    <main>
      <div class="video-card">
        <VideoCard />
      </div>
    </main>
  </div>
</template>

<script lang="ts">
import VideoCard from '@/components/VideoCard.vue';
import ThemeToggle from '@/components/ThemeToggle.vue';

export default {
  name: 'HomeView',
  components: {
    VideoCard,
    ThemeToggle,
  },
};

</script>

<style scoped>
.home {
  /* span the full width of the app grid so header and main stack vertically */
  grid-column: 1 / -1;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  width: 100%;
  text-align: left;
  margin-top: 30px;
}

h1 {
  text-align: left;
}

.header-tools {
  display: flex;
  align-items: center;
}

main {
  width: 100%;
  display: flex;
  justify-content: center;
}

.video-card {
  margin: 5px;
  width: 100%;
  max-width: 960px;
}

</style>
