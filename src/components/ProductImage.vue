<template>
    <div class="flex flex-col md:flex-row md:w-full md:h-full justify-center">
        <div class="flex flex-col relative md:w-1/2 bg-white py-6 max-w-[600px]">
            <div class="relative flex justify-between items-center">
                <button class="text-3xl p-1" @click="prevSlide">&lt;</button>
                <div class="flex justify-center relative mr-2 ml-2" @click="openModal" ref="imageBox">
                    <!-- main image desktop -->
                    <div v-if="!isMediumWidth() && media[currentSlide].type === 'image'" class="relative inline-block"
                        @mousemove="handleMouseMove"
                        @mouseleave="handleMouseLeave"
                    >
                        <img :src="media[currentSlide].source" alt="Product Image" class="h-[490px] object-contain" ref="image">
                    </div>
                    <div v-else-if="media[currentSlide].type === 'video'" class="relative inline-block">
                        <video :src="media[currentSlide].source" alt="Product Video" class="h-[490px] object-contain" ref="image" controls></video>
                    </div>
                    <!-- main image mobile -->
                    <div v-else class="overflow-hidden relative w-full touch-action-pan-y">
                        <div class="flex transition-transform ease-in-out duration-300 items-center" :style="{ 'transform': `translateX(${translateX}px)` } "
                        @touchstart="onTouchStart"
                        @touchmove="onTouchMove"
                        @touchend="onTouchEnd">
                        <div v-for="(image, index) in media" :key="index" class="flex-none w-full flex justify-center">
                            <img :src="image.source" alt="Selected Image" class="h-[490px] object-contain" >
                        </div>
                        </div>
                    </div>
                </div>
                <button class="text-3xl p-1" @click="nextSlide">&gt;</button>
            </div>
            <!-- indicator circles -->
            <div class="absolute bottom-[-20px] left-0 right-0 flex justify-center">
                <div v-for="(image, index) in media" :key="index" class="w-3 h-3 rounded-full mx-1 bg-gray-400 cursor-pointer"
                    :class="{ 'bg-orange-500': currentSlide === index }" @click="setCurrentSlide(index)">
                </div>
            </div>

            <!-- thumbnails -->
            <div class="flex justify-center mt-4">
                <div  v-for="(image, index) in media" :key="index" class="md:block hidden">
                    <div v-if="image.type === 'image'">
                        <img class="h-[60px] object-contain ml-3 mr-3" :class="{ ' border-2 border-black': currentSlide === index}" :src="image.thumbSource" @click="setCurrentSlide(index)"/>
                    </div>
                    <div v-else-if="image.type === 'video'" class="relative">
                        <video class="h-[60px] object-contain ml-3 mr-3" :class="{ ' border-2 border-black': currentSlide === index }" :src="image.source" @click="setCurrentSlide(index)"></video>
                        <div class="absolute top-0 left-0 w-full h-full flex items-center justify-center opacity-80" @click="setCurrentSlide(index)">
                            <div class="bg-white rounded-full p-1 w-1/2">
                                <svg xmlns="http://www.w3.org/2000/svg" class=" text-black" viewBox="0 0 20 20" fill="currentColor">
                                <path d="M8 5v10l9-5-9-5z" />
                                </svg>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!-- zoomed view -->
        <div class="md:w-1/2 mt-2 max-w-[600px]" ref="zoomedDiv">
            <div v-if="showZoomBox && media[currentSlide].type === 'image'" 
                class="ml-4 bg-white border border-gray-300 shadow-lg z-10 overflow-hidden w-full max-h-[80vh] ">
                <img :src="media[currentSlide].largeSource" alt="Zoomed Image" class="w-full object-contain overflow-hidden" ref="zoomedBox"
                    :style="{ transform: `scale(${zoomScale})`, 'transform-origin': zoomOrigin }" />
            </div>
            <div v-else>
                <ProductDescription class="md:pl-8"/>
            </div>
        </div>
    </div>
    <MediaModal :media="media" v-if="showModal" @close="closeModal" />
</template>

<script>
import MediaModal from './MediaModal.vue';
import ProductDescription from './ProductDescription.vue';
export default {
    name: 'ProductImage',
    components: {
        MediaModal,
        ProductDescription
    },
    data() {
        return {
            showZoomBox: false,
            zoomScale: 2,
            zoomOrigin: '',
            currentSlide: 0,
            zoomRadius: 0,
            showModal: false,
            touchStartX: 0,
            touchEndX: 0,
            touchThreshold: 60,
            translateX: 0,
            startX: 0,
            currentX: 0,
            isDragging: false,
            zoomX: 0,
            zoomY:0,
            media: [
                { type: 'image', source: '/media/product-image.jpeg', largeSource: '/media/product-image-large.jpeg', thumbSource: '/media/product-image-thumbnail.jpeg' },
                { type: 'image', source: '/media/product-image-back.jpeg', largeSource: '/media/product-image-back-large.jpeg', thumbSource: '/media/product-image-back-thumbnail.jpeg' },
                { type: 'image', source: '/media/product-image-front.jpeg', largeSource: '/media/product-image-front-large.jpeg', thumbSource: '/media/product-image-front-thumbnail.jpeg' },
                {type: 'video', source: '/media/drag-swipe-in-mobile.mov', },
                { type: 'video', source: '/media/double-tap-zoom-mobile.mov', },
            ],
        };
    },
    mounted() {
        this.media = this.isMediumWidth() ? this.media.filter(item => item.type === 'image') : this.media
    },
    methods: {
        prevSlide() {
            const image = this.$refs.imageBox;
            const rect = image.getBoundingClientRect();
            this.currentSlide = (this.currentSlide - 1 + this.media.length) % this.media.length;
            if(this.currentSlide === this.media.length-1){
                this.translateX -= rect.width*(this.media.length-1)
            } else {
                this.translateX += rect.width;
            }
            
        },
        nextSlide() {
            const image = this.$refs.imageBox;
            const rect = image.getBoundingClientRect();
            this.currentSlide = (this.currentSlide + 1) % this.media.length;
            if(this.currentSlide === 0){
                this.translateX += rect.width*(this.media.length-1)
            } else {
                this.translateX -= rect.width;
            }
           
        },
        handleMouseMove(event) {
            if (this.isMediumWidth()) {
                return;
            }
            this.showZoomBox = true;
            const image = this.$refs.image;
            if(!this.$refs.zoomedBox || !this.$refs.zoomedDiv)
                return;
            const zoomedBox = this.$refs.zoomedBox.getBoundingClientRect();
            const zoomedDiv = this.$refs.zoomedDiv.getBoundingClientRect();
            const rect = image.getBoundingClientRect();
            const zoomImageOverflowYScale = ((zoomedBox.height/this.zoomScale)/zoomedDiv.height);
            const scaleX = ((zoomedBox.width / this.zoomScale) / rect.width);
            const scaleY = ((zoomedBox.height/this.zoomScale)* zoomImageOverflowYScale) / image.height;
            const originalOffsetX = event.clientX - rect.left;
            const originalOffsetY = event.clientY - rect.top;
            const distanceFromMiddleX = Math.abs(originalOffsetX - (rect.width / 2));
            const radiusX = originalOffsetX < (rect.width/2) ? - (distanceFromMiddleX /this.zoomScale) : (distanceFromMiddleX / this.zoomScale);
            const distanceFromMiddleY = Math.abs(originalOffsetY - (rect.height / 2));
            const radiusY = originalOffsetY < (rect.height / 2) ? - (distanceFromMiddleY / this.zoomScale) : (distanceFromMiddleY / this.zoomScale);
            const offsetX = originalOffsetX *scaleX + radiusX;
            const offsetY = originalOffsetY *scaleY + radiusY;
            this.zoomX = offsetX;
            this.zoomY = offsetY;

            this.zoomOrigin = `${this.zoomX}px ${this.zoomY}px`;
        },
        setCurrentSlide(val) {
            if(!this.isMediumWidth()){
                this.currentSlide = val;
                return;
            }
            if (val === this.currentSlide)
                return;
            if (val < this.currentSlide) {
                while (this.currentSlide !== val) {
                    this.prevSlide()
                }
            } else {
                while (this.currentSlide !== val) {
                    this.nextSlide()
                }
            }
        },
        handleMouseLeave() {
            if (this.isMediumWidth()) {
                return;
            }
            this.showZoomBox = false;
        },
        openModal() {
            this.showModal = true;
        },
        closeModal() {
            this.showModal = false;
        },
        isMediumWidth() {
            return window.innerWidth < 769;
        },
        onTouchStart(event) {
            this.isDragging = true;
            this.startX = event.touches[0].clientX;
            this.currentX = this.translateX;
        },
        onTouchMove(event) {
            if (!this.isDragging) return;

            const touch = event.touches[0];
            const deltaX = touch.clientX - this.startX;
            this.currentX = this.translateX + deltaX;
        },
        onTouchEnd() {
            if (!this.isDragging) return;

            this.isDragging = false;
            const threshold = 50;

            if (this.currentX - this.translateX > threshold && this.translateX < 0) {
                this.prevSlide();
            } else if (this.translateX - this.currentX > threshold && this.translateX > -(window.innerWidth * (this.media.length - 1))) {
                this.nextSlide();
            }
            this.currentX = this.translateX;
        },
    },
};
</script>

