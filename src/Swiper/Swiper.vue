<template>
    <div ref="swiper" 
         class="swiper-container" 
         :style="{width: `${width/75}rem`, height: `${height/75}rem`}">
        <div ref="swiperList" 
             class="swiper-wrap" 
             :style="{transform: `translate3d(${(-width*(current+1)+offset)/75}rem, 0 , 0)`, transition: `${isScroll ? 'transform': 'none'} ${speed}ms`}"
             @transitionend="onTransitionEnd">
            <slot></slot>
        </div>
        <div class="icon"
             v-if="$slots['prev-icon']" 
             :style="prevStyle" 
             @click.stop="prev" 
             @touchstart.stop="" 
             @touchmove.stop="" 
             @touchend.stop="">
            <slot name="prev-icon"></slot>
        </div>
        <div class="icon"
             v-if="$slots['next-icon']" 
             :style="nextStyle" 
             @click.stop="next" 
             @touchstart.stop="" 
             @touchmove.stop="" 
             @touchend.stop="">
            <slot name="next-icon"></slot>
        </div>
        <ul v-if="dots" class="dots-list">
            <li class="dot" 
                v-for="index in length"
                :key="index" 
                :style="{backgroundColor: index - 1 === realCurrent ? dotsActiveColor : dotsColor}">
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    name: 'swiper',
    props: {
        // swiper宽度
        width: {
            type: Number,
            default: () => 650
        },
        // swiper长度
        height: {
            type: Number,
            default: () => 300
        },
        // 切换间隔
        interval: {
            type: Number,
            default: () => 8000
        },
        // 切换速度
        speed: {
            type: Number,
            default: () => 300
        },
        // 显示面板指示点
        dots: {
            type: Boolean,
            default: () => false
        },
        // 指示点未选中颜色
        dotsColor: {
            type: String,
            default: () => 'rgba(0, 0, 0, .3)'
        },
        // 指示点选中颜色
        dotsActiveColor: {
            type: String,
            default: () => '#fff'
        },
        // prev按钮的位置大小信息对象
        prevStyle: {
            type: Object,
            default: () => ({left: '5%', top: '40%', height: '20%', width: '5%'})
        },
        // next按钮的位置大小信息对象
        nextStyle: {
            type: Object,
            default: () => ({right: '5%', top: '40%', height: '20%', width: '5%'})
        }
    },
    data: () => ({
        current: 0, // 当前轮播到的索引号（包括前后添加的两张图）
        length: 0, // 实际的轮播数量
        offset: 0, // x轴偏移量
        isScroll: false, // 是否正在切换
        isTouching: false,
        firstMove: '', // 每次touch事件第一次发生轴线变换的方向
        timer: null, // 定时器
        touchInfo: {} // 第一次触摸点的信息
    }),
    computed: {
        realCurrent() {
            // 实际当前轮播到的索引号（不包括前后添加的两张图）
            return this.current < 0 ? 2 : this.current >= this.length ? this.length - 1 : this.current;
        }
    },
    mounted() {
        this.length = this.$refs.swiperList.children.length;
        
        const swiperListFirstChild = this.$refs.swiperList.children[0];
        const swiperListLastChild = this.$refs.swiperList.children[this.length - 1];
        this.$refs.swiperList.appendChild(swiperListFirstChild.cloneNode(true));
        this.$refs.swiperList.insertBefore(swiperListLastChild.cloneNode(true), swiperListFirstChild);

        this.$refs.swiper.addEventListener('touchstart', this.onTouchStart);
        this.$refs.swiper.addEventListener('touchmove', this.onTouchMove);
        this.$refs.swiper.addEventListener('touchend', this.onTouchEnd);
        this.$refs.swiper.addEventListener('touchcancel', this.onTouchEnd);

        this.setTimer();
    },
    beforeDestroy() {
        this.$refs.swiper.removeEventListener('touchstart', this.onTouchStart);
        this.$refs.swiper.removeEventListener('touchmove', this.onTouchMove);
        this.$refs.swiper.removeEventListener('touchend', this.onTouchEnd);
        this.$refs.swiper.removeEventListener('touchcancel', this.onTouchEnd);
        this.clearTimer();
    },
    methods: {
        onTouchStart(e) {
            this.isTouching = true;
            this.clearTimer();
            this.touchInfo = e.targetTouches[0];
        },
        onTouchMove(e) {
            let startX = this.touchInfo.clientX;
            let startY = this.touchInfo.clientY;
            let lastX = e.targetTouches[0].clientX;
            let lastY = e.targetTouches[0].clientY;
            if (this.firstMove === '') {
                if (Math.abs(lastX - startX) > 20) {
                    e.preventDefault();
                    this.firstMove = 'x';
                    this.offset = (lastX - startX) * 2 / 3;
                } else if (Math.abs(lastY - startY) > 20) {
                    this.firstMove = 'y';
                }
            } else if (this.firstMove === 'x') {
                e.preventDefault();
                this.offset = (lastX - startX) * 2 / 3;
            }
        },
        onTouchEnd(e) {
            if (this.firstMove === 'x') {
                Math.abs(this.offset) >= this.width / 10 && (this.offset > 0 ? this.current-- : this.current++);
                this.offset = 0;
                this.isScroll = true;
            } else {
                this.setTimer();
            }
            this.isTouching = false;
            this.firstMove = '';
            this.touchInfo = {};
        },
        onTransitionEnd() {
            this.isScroll = false;
            this.current = this.current < 0 ? this.length - 1 : this.current >= this.length ? 0 : this.current;
            !this.isTouching && this.setTimer();
        },
        prev() {
            if (this.isScroll) return;
            this.clearTimer();
            this.isScroll = true;
            this.current--;
        },
        next() {
            if (this.isScroll) return;
            this.clearTimer();
            this.isScroll = true;
            this.current++;
        },
        setTimer() {
            this.timer = setTimeout(() => this.next(), this.interval);
        },
        clearTimer() {
            clearTimeout(this.timer);
        }
    }
}
</script>

<style lang="scss" scoped>
    .swiper-container {
        position: relative;
        left: 0;
        top: 0;
        margin: 0 auto;
        overflow: hidden;
        .swiper-wrap {
            height: 100%;
            width: 100%;
            white-space: nowrap;
            font-size: 0;
        }
        .dots-list {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            white-space: nowrap;
            z-index: 10;
            .dot {
                display: inline-block;
                height: 12px;
                width: 12px;
                margin: 0 8px;
                border-radius: 50%;
            }
        }
        .icon {
            position: absolute;
            z-index: 10;
        }
    }
</style>

