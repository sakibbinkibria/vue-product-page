<template>
    <div class="flex flex-col px-5 mt-3 md:mt-0">
        <div class="flex justify-between">
            <span class="text-gray-500">{{ company }}</span>
            <span class="text-green-600">{{ stockText }}</span>
        </div>
        
        <h2 class="text-2xl font-semibold">{{ model }}</h2>
        <span class="mt-4">£{{ price }}</span>
        <p class="border-b-2 border-gray-400 w-full my-5"></p>
        <div v-for="(data, index) in details" :key="index">
            <div class="flex">
                <span class="font-semibold mr-2">{{ data.name }}:</span>
                <span>{{ data.value }}</span>
            </div>
        </div>
        <div class="flex my-2 items-center">
            <button class="font-bold text-gray-500 text-2xl inline-flex items-center" @click="decreaseQuantity" :disabled="quantity === 0"> - </button>
            <input class="bg-gray-300 w-10 text-center mx-2" :value="quantity">
            <button class="font-bold text-gray-500 text-2xl" @click="increaseQuantity"> + </button>
        </div>
        <button class="my-3 bg-yellow-500 w-full py-2 font-semibold">Add to Basket</button>
    </div>
</template>

<script>
export default {
    name: 'ProductDescription',
    data() {
        return {
            company: 'Samsung',
            model: 'Samsung Galaxy',
            price: 880,
            sim: 'DUAL SIM',
            stockText: 'Hurry stock limited',
            quantity: 1,
            details: [
                {
                    name: 'Sim',
                    value: 'DUAL SIM'
                },
                {
                    name: 'Condition',
                    value: 'LIKE NEW'
                },
                {
                    name: 'Colour',
                    value: 'LAVENDER'
                },
                {
                    name: 'Network',
                    value: 'UNLOCKED'
                },
                {
                    name: 'Storage',
                    value: '512GB'
                },
            ]

        }
    },
    methods : {
        increaseQuantity() {
            this.quantity = this.quantity +1;
        },
        decreaseQuantity() {
            if (this.quantity === 0)
                return;
            this.quantity = this.quantity - 1;
        }
    }
};
</script>
