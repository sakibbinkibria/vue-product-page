<template>
    <div class="fixed top-0 left-0 w-screen h-screen flex items-center justify-center bg-gray-900 bg-opacity-50 z-50">
        <div class="bg-white w-full h-full md:w-5/6 md:h-5/6 flex flex-col relative p-0 md:p-[40px]">
            <!-- tabs -->
            <div v-if="!isMediumWidth()" class="p-1 mb-3">
                <button @click="switchTab('image')" :class="{ 'border-b-2 border-gray-600': selectedTab === 'image' }" class="mr-2">Images</button>
                <button @click="switchTab('video')" :class="{ 'border-b-2 border-gray-600': selectedTab === 'video' }">Videos</button>
            </div>
            <div v-else>
                <button @click="closeModal" class="m-3 border-2 bg-white shadow-md px-3 py-1 rounded-md">Back</button>
            </div>
            <!-- contents -->
            <div class="max-w-full h-full">
                <!-- close button -->
                <div  v-if="!isMediumWidth()" class="absolute top-2 right-2 p-2">
                    <button @click="closeModal" class="text-gray-600 hover:text-black focus:outline-none">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M6.293 6.293a1 1 0 011.414 0L10 8.586l2.293-2.293a1 1 0 111.414 1.414L11.414 10l2.293 2.293a1 1 0 11-1.414 1.414L10 11.414l-2.293 2.293a1 1 0 01-1.414-1.414L8.586 10 6.293 7.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                        </svg>
                    </button>
                </div>
                <div class="flex flex-col md:flex-row items-center md:items-start md:justify-between w-full h-full">
                    <!-- main image -->
                    <div class="md:h-auto flex flex-col justify-between items-center md:flex-row md:justify-center md:w-3/4 relative ">
                        <div  v-if="selectedTab === 'image'" class="flex items-center justify-center">
                            <div v-if="isMediumWidth()" class="overflow-hidden relative w-full touch-action-pan-y">
                                <div class="flex transition-transform ease-in-out duration-300 items-center" :style="{ 'transform': `translateX(${translateX}px)` }"
                                @touchstart="onTouchStart"
                                @touchmove="onTouchMove"
                                @touchend="onTouchEnd">
                                <div v-for="(image, index) in filteredMedia" :key="index" class="flex-none w-full carousel-slide">
                                    <img
                                        :src="zoomedIn ? image.largeSource : image.source"
                                        alt="Selected Image"
                                        class="w-screen max-h-[80vh] md:max-h-[60vh] md:w-auto object-contain"
                                        :style="{ transform: currentIndex === index && zoomedIn ? zoomTransform : '', 'transform-origin': currentIndex === index ? this.zoomOrigin : '', transition: currentIndex === index ? 'transform ease .5s' : '' }"
                                        @click="handleClick"
                                    />
                                </div>
                                </div>
                            </div>
                            <div v-else class="overflow-hidden">
                                <img :src="zoomedIn ? selectedMedia.largeSource : selectedMedia.source" alt="Selected Image" class="w-screen max-h-[80vh] md:max-h-[60vh] md:w-auto object-contain"
                                :style="{ transform: zoomedIn ? zoomTransform : '', 'transform-origin': this.zoomOrigin, transition: 'transform ease .5s' }" @click="handleClick">
                            </div>
                        
                        </div>
                        <div v-else-if="selectedTab === 'video'" class="flex items-center justify-center">
                            <video :src="selectedMedia.source" controls class="max-h-[500px] md:max-h-[60vh]"></video>
                        </div>
                        <!-- indicator circles -->
                        <div v-if="isMediumWidth()" class="absolute bottom-4 left-0 right-0 flex justify-center">
                            <div v-for="(image, index) in filteredMedia" :key="index" class="w-3 h-3 rounded-full mx-1 bg-gray-400 cursor-pointer"
                                :class="{ 'bg-orange-500': currentIndex === index }" @click="selectMedia(index)">
                            </div>
                        </div>
                    </div>

                    <!-- horizontal thumbnails -->
                    <div class="p-2 border-2 w-[fit-content] md:hidden block mt-2">
                            <div class="flex md:flex-col">
                                <div v-for="(item, index) in filteredMedia" :key="index"
                                    :class="{ ' border-2 border-black': currentIndex === index, 'mr-2': index !== filteredMedia.length - 1 }">
                                    <img v-if="item.type === 'image'" :src="item.thumbSource" alt="Thumbnail" @click="selectMedia(index)"
                                        class="h-[60px] object-cover cursor-pointer border border-gray-300">
                                    <video v-else-if="item.type === 'video'" :src="item.source" @click="selectMedia(index)"
                                        class="h-[60px] object-cover cursor-pointer border border-gray-300"></video>
                                </div>
                            </div>
                        </div>
                    <div class="p-1 md:block hidden w-1/4">
                        <div class="flex flex-col">
                            <h3>Product details</h3>
                            <!-- thumbnails in right side -->
                            <div class="p-2  md:block hiden mt-2">
                                <div class="flex flex-wrap">
                                    <div v-for="(item, index) in filteredMedia" :key="index" class="mt-2"
                                        :class="{ ' border-2 border-black': item.source === selectedMedia.source, 'mr-2': index !== filteredMedia.length - 1 }">
                                        <img v-if="item.type === 'image'" :src="item.thumbSource" alt="Thumbnail" @click="selectMedia(index)"
                                            class="h-[60px] object-cover cursor-pointer border border-gray-300">
                                        <video v-else-if="item.type === 'video'" :src="item.source" @click="selectMedia(index)"
                                            class="h-[60px] object-cover cursor-pointer border border-gray-300"></video>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
    </div>
</template>

<script>
export default {
    props: {
        media: {
            type: Array,
            required: true,
        },
    },
    data() {
        return {
            currentIndex: 0,
            selectedMedia: this.media[0],
            zoomedIn: false,
            zoomTransform: 'scale(2)',
            selectedTab: 'image',
            filteredMedia: this.filterMedia('image'),
            touchStartX: 0,
            touchEndX: 0,
            touchThreshold: 60,
            zoomOrigin: '',
            translateX: 0,
            startX: 0,
            currentX: 0,
            isDragging: false,
        };
    },
    methods: {
        filterMedia(tab) {
            const array =  this.media.filter((item) => item.type === tab);
            return array;
        },
        selectMedia(index) {
            if (!this.isMediumWidth()){
                this.selectedMedia = this.filteredMedia[index];
                return;
            }
            if(index === this.currentIndex)
                return;
            if(index < this.currentIndex) {
                while (this.currentIndex !== index) {
                    this.prevSlide()
                }
            } else {
                while (this.currentIndex !== index) {
                    this.nextSlide()
                }
            }
        },
        closeModal() {
            this.$emit('close');
        },
        toggleZoom(event) {
            if (this.zoomedIn) {
                this.zoomOut()
            } else {
                this.zoomImage(event)
            }
        },
        zoomImage(event) {
            this.zoomedIn = true;
            const image = event.target;
            const rect = image.getBoundingClientRect();
            const offsetX = event.clientX - rect.left;
            const offsetY = event.clientY - rect.top;

            const scaleX = image.width / rect.width;
            const scaleY = image.height / rect.height;
            this.zoomRadius = 0;

            const zoomX = Math.min(Math.max(offsetX * scaleX - this.zoomRadius, 0), rect.width * scaleX - this.zoomRadius);
            const zoomY = Math.min(Math.max(offsetY * scaleY - this.zoomRadius, 0), rect.height * scaleY - this.zoomRadius);
            this.zoomOrigin = `${zoomX}px ${zoomY}px`;
        },
        zoomOut() {
            this.zoomedIn = false;
        },
        switchTab(tab) {
            this.selectedTab = tab;
            this.filteredMedia = this.filterMedia(tab);
            this.selectedMedia = this.filteredMedia[0];
        },
        isMediumWidth() {
            return window.innerWidth < 769;
        },
        handleDoubleTap(event) {
            let now = new Date().getTime();
            let lastTouch = this.lastTouch || now + 1;
            let delta = now - lastTouch;
            if (delta < 500 && delta > 0) {
                // Double tap detected
                this.lastTouch = null;
                this.toggleZoom(event);
            } else {
                // Single tap detected
                this.lastTouch = now;
            }
        },
        handleClick(event) {
            if(this.isMediumWidth()){
                this.handleDoubleTap(event);
            } else {
                this.toggleZoom(event);
            }
        },
        prevSlide() {
            this.currentIndex = (this.currentIndex - 1 + this.filteredMedia.length) % this.filteredMedia.length;
            this.translateX += window.innerWidth;
        },
        nextSlide() {
            this.currentIndex = (this.currentIndex + 1) % this.filteredMedia.length;
            this.translateX -= window.innerWidth;
        },
        onTouchStart(event) {
            if (this.zoomedIn || this.isDragging) return;
            this.isDragging = true;
            this.startX = event.touches[0].clientX;
            this.currentX = this.translateX;
        },
        onTouchMove(event) {
            if (!this.isDragging || this.zoomedIn) return;

            const touch = event.touches[0];
            const deltaX = touch.clientX - this.startX;
            this.currentX = this.translateX + deltaX;
        },
        onTouchEnd() {
            if (!this.isDragging || this.zoomedIn) return;

            this.isDragging = false;
            const threshold = 70; // Set a threshold to determine if swipe should change slide

            if (this.currentX - this.translateX > threshold && this.translateX < 0) {
                this.prevSlide();
            } else if (this.translateX - this.currentX > threshold && this.translateX > -(window.innerWidth * (this.filteredMedia.length - 1))) {
                this.nextSlide();
            }

            // Reset position if swipe threshold is not met
            this.currentX = this.translateX;
        },
    },
};
</script>

<style scoped>/* Your styles go here */</style>
