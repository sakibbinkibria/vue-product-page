<template>
  <div class="flex flex-col bg-gray-200">
    <NavBar/>
    <div class="flex justify-center p-2 md:p-10 min-h-[calc(100vh-60px)] items-start">
      <ProductImage />
    </div>
    
  </div>
</template>

<script>
import ProductImage from './components/ProductImage.vue';
import NavBar from './components/NavBar.vue';
import './main.css';

export default {
  name: 'App',
  components: {
    ProductImage,
    NavBar
  },
};
</script>
