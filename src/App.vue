<template>
  <div class="relative min-h-screen bg-cover bg-center" :style="backgroundStyle">
    <div class="bg-black/40 absolute inset-0 z-0"></div> <!-- optional dark overlay -->
    <div class="relative z-10">
      <Quiz />
    </div>
  </div>
</template>

<script>
import Quiz from './components/Quiz.vue';
import bgImage from './assets/ghana.jpeg'; 

export default {
  name: 'App',
  components: {
    Quiz,
  },
  computed: {
    backgroundStyle() {
      return {
        backgroundImage: `url(${bgImage})`,
        backgroundSize: 'cover',
        backgroundPosition: 'center',
      };
    },
  },
};
</script>

<style>
body {
  margin: 0;
}
</style>
