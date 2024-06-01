<!-- GetPrice.vue -->
<template>
  <div class="price-container">
    <div class="hello">Hello, {{ stockSymbol }}</div>
    <div class="price-info">
      <span class="last-price">{{ lastPrice }}</span>
      <span :class="priceChange > 0 ? 'price-up' : 'price-down'">
        {{ priceChange }} ({{ priceChangePercentage }}%)
      </span>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref, computed, onMounted } from "vue";

export default {
  name: "GetPrice",
  props: {
    stockSymbol: {
      type: String,
      required: true,
    },
  },
  setup(props) {
    const lastPrice = ref(0);
    const previousClose = ref(0);
    const priceChange = ref(0);

    onMounted(() => {
      axios
        .get(
          `https://api.fugle.tw/marketdata/v1.0/stock/intraday/quote/${props.stockSymbol}`,
          {
            headers: {
              "X-API-KEY": process.env.VUE_APP_API_KEY,
            },
          },
        )
        .then((res) => {
          lastPrice.value = res.data["lastPrice"];
          previousClose.value = res.data["previousClose"];
          priceChange.value = (res.data['lastPrice'] - res.data['previousClose']).toFixed(2);
          console.log(res.data["lastPrice"]);
        });
    });

    const priceChangePercentage = computed(() => {
      if (previousClose.value === 0) return 0;
      return ((priceChange.value / previousClose.value) * 100).toFixed(2);
    });

    return {
      lastPrice,
      priceChange,
      priceChangePercentage,
    };
  },
};
</script>

<style scoped>
.price-container {
  border: 1px solid #ccc;
  padding: 20px;
  margin: 10px;
  text-align: center;
  width: 200px;
  height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.hello {
  margin-bottom: 10px;
}

.price-info {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
}

.last-price {
  color: black;
  margin-right: 10px;
}

.price-up {
  color: red;
}

.price-down {
  color: green;
}
</style>
