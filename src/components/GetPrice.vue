<template>
  <div class="hello">Hello, 2330</div>
  <div class="price-info">
    <span class="last-price">{{ lastPrice }}</span>
    <span :class="priceChange > 0 ? 'price-up' : 'price-down'">
      {{ priceChange }} ({{ priceChangePercentage }}%)
    </span>
  </div>
</template>

<script>
import axios from "axios";
import { ref, computed, onMounted } from "vue";

export default {
  name: "GetPrice",
  setup() {
    const lastPrice = ref(0);  // 使用 ref 创建响应式数据
    const previousClose = ref(0);

    onMounted(() => {
      axios
        .get("https://api.fugle.tw/marketdata/v1.0/stock/intraday/quote/2330", {
          headers: {
            "X-API-KEY": process.env.VUE_APP_API_KEY,
          },
        })
        .then((res) => {
          lastPrice.value = res.data['lastPrice'];  // 使用 .value 访问 ref 的值
          previousClose.value = res.data['previousClose'];
          console.log(res.data['lastPrice']);
        });
    });

    const priceChange = computed(() => lastPrice.value - previousClose.value);
    const priceChangePercentage = computed(() => {
      if (previousClose.value === 0) return 0;
      return ((priceChange.value / previousClose.value) * 100).toFixed(2);
    });

    return {
      lastPrice,  // 返回响应式数据以便在模板中使用
      priceChange,
      priceChangePercentage,
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  text-align: center;
  margin-bottom: 20px;
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

h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
