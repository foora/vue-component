<template>
    <main>
        <div class="img_modal">
            <img class="image is-96x96" :src="img_modal_src">
        </div>
        <input type="file" class="hide" @change="fileChange">
        <a class="button is-primary button_width" @click="upload">选择图片</a>
    </main>
</template>
<script>
    const emptyImg =
        "data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==";
    export default {
        props: {
            imgUrl: {
                type: String,
                default () {
                    return emptyImg
                },
            }
        },
        data() {
            return {
                img_modal_src: this.imgUrl ? this.imgUrl : emptyImg
            }
        },
        methods: {
            upload(e) {
                e.target.previousElementSibling.click();
            },
            fileChange(e) {
                let file = e.target.files[0];
                if (file) {
                    if (file.type !== "image/jpeg" && file.type !== "image/png" && file.type !== "image/jpg" && file.type !==
                        "image/gif") {
                        this.$emit("error", "上传所选文件格式不正确");
                        return;
                    }
                    URL.revokeObjectURL(this.img_modal_src);
                    this.img_modal_src = URL.createObjectURL(file);
                    this.$emit("needUpLoad", file);
                } else {
                    this.reset();
                }
            },
            reset() {
                URL.revokeObjectURL(this.img_modal_src);
                this.img_modal_src = this.imgUrl ? this.imgUrl : emptyImg;
                this.$emit("needUpLoad");
            }
        }
    }
</script>

<style scoped>
    .hide {
        display: none;
    }

    .button_width {
        width: 104px;
    }

    .img_modal {
        box-sizing: border-box;
        padding: 4px;
        height: 104px;
        width: 104px;
        border: 1px solid grey;
    }
</style>