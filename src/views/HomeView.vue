<script setup>
  import { ref, onMounted } from "vue"
  import heroImage from "../assets/images/heroImage.png"
  import gsap from "gsap"
  import BarBuilder from "../components/builder/BarBuilder.vue"

  let hero_img_url = heroImage

  function source(src) {
    const baseUrl = location.protocol + "//" + location.host + '/';
    return baseUrl + src;
  }
</script>

<template>
  <div class="bg-gradient-to-r flex justify-center flex-row flex-wrap items-center w-full lg:flex-row min-h-screen relative" id="home-wrapper">
    <BarBuilder />
  </div>
</template>
