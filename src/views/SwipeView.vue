<template>
    <swiper :slides-per-view="1" navigation pagination>
        <swiper-slide v-for="(slide, index) in slides" :key="index">
            Slide {{ index + 1 }}
        </swiper-slide>
    </swiper>
</template>

<script setup>
import { ref } from 'vue';
import { Swiper, SwiperSlide } from 'swiper/vue';

const slides = ref(['Slide 1', 'Slide 2', 'Slide 3']);
</script>

<style scoped>
/* 추가적인 스타일 설정 가능 */
</style>