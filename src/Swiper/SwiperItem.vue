<template>
    <div class="swiper-item">
        <slot></slot>
    </div>
</template>

<script>
export default {
    name: 'swiper-item'
}
</script>

<style lang="scss" scoped>
    .swiper-item {
        display: inline-block;
        width: 100%;
        height: 100%;
    }
</style>
