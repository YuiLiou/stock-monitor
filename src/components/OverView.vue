<!-- OverView.vue -->
<template>
  <div class="overview">
    <div class="input-container">
      <input v-model="newStockSymbol" placeholder="Enter stock symbol" />
      <button @click="addStock">Add Stock</button>
    </div>
    <div class="grid-container">
      <GetPrice
        v-for="(stock, index) in stocks"
        :key="index"
        :stockSymbol="stock"
      />
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import GetPrice from "./GetPrice.vue";

export default {
  components: {
    GetPrice,
  },
  setup() {
    const newStockSymbol = ref("");
    const stocks = ref(["2330", "8299", "2501", "2527"]);

    const addStock = () => {
      if (newStockSymbol.value.trim() !== "") {
        stocks.value.push(newStockSymbol.value.trim());
        newStockSymbol.value = "";
      }
    };

    return {
      newStockSymbol,
      stocks,
      addStock,
    };
  },
};
</script>

<style scoped>
.overview {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.input-container {
  margin-bottom: 20px;
}

.grid-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

input {
  margin-right: 10px;
  padding: 5px;
}
</style>
