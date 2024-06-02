<template>
  <div
    class="price-container"
    :class="{ hovered: isHovered }"
    @mouseover="isHovered = true"
    @mouseout="isHovered = false"
  >
    <div class="name" :class="{ hovered: isHovered }">
      {{ name }} ({{ stockSymbol }})
    </div>
    <div class="price-info" :class="{ hovered: isHovered }" :style="stockStyle">
      <span class="last-price">{{ lastPrice }}</span>
      <span>
        <template v-if="priceChange > 0">⬆</template>
        <template v-else-if="priceChange < 0">⬇</template>
        {{ Math.abs(priceChange) }} ({{ priceChangePercentage }}%)
      </span>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref, onMounted, onUnmounted, computed } from "vue";

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
    const priceDownOpacity = ref(1);
    const priceUpOpacity = ref(1);
    const name = ref("");
    const isHovered = ref(false);

    const fetchStockPrice = async () => {
      try {
        const res = await axios.get(
          `https://api.fugle.tw/marketdata/v1.0/stock/intraday/quote/${props.stockSymbol}`,
          {
            headers: {
              "X-API-KEY": process.env.VUE_APP_API_KEY,
            },
          },
        );
        lastPrice.value = res.data["lastPrice"];
        previousClose.value = res.data["previousClose"];
        priceChange.value = (
          res.data["lastPrice"] - res.data["previousClose"]
        ).toFixed(2);
        name.value = res.data["name"];
        console.log(priceChange.value);
      } catch (error) {
        console.error("Error fetching stock price:", error);
      }
    };

    const stockStyle = computed(() => {
      return {
        opacity:
          priceChange.value < 0
            ? priceDownOpacity.value
            : priceChange.value > 0
              ? priceUpOpacity.value
              : 1,
        boxShadow:
          priceChange.value < 0 ? "1px 1px 10px #b5ffae" : "1px 1px 10px red",
        color: priceChange.value < 0 ? "green" : "red",
      };
    });
    let priceDownInterval, priceUpInterval;
    onMounted(() => {
      priceDownInterval = setInterval(() => {
        if (priceDownOpacity.value > 0.4) {
          priceDownOpacity.value = priceDownOpacity.value - 0.1;
        } else {
          priceDownOpacity.value = 1;
        }
      }, 140);

      priceUpInterval = setInterval(() => {
        if (priceUpOpacity.value > 0.4) {
          priceUpOpacity.value = priceUpOpacity.value - 0.1;
        } else {
          priceUpOpacity.value = 1;
        }
      }, 140);
    });

    onUnmounted(() => {
      clearInterval(priceDownInterval);
      clearInterval(priceUpInterval);
    });

    onMounted(async () => {
      fetchStockPrice();
    });

    const priceChangePercentage = computed(() => {
      if (previousClose.value === 0) return 0;
      return ((priceChange.value / previousClose.value) * 100).toFixed(2);
    });

    return {
      lastPrice,
      priceChange,
      priceChangePercentage,
      stockStyle,
      priceDownOpacity,
      priceUpOpacity,
      name,
      isHovered,
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
  border-radius: 10px;
  font-family: "Microsoft JhengHei", sans-serif;
  font-weight: bold;
}

.price-container.hovered {
  opacity: 0.7;
  background-color: #b5b5b5;
  color: white;
}

.name {
  margin-bottom: 10px;
  font-size: 24px;
  color: #191970;
}

.name.hovered {
  color: white;
}

.price-info {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  opacity: 1;
  transition: opacity 0.2s ease-in-out;
}

.price-info.hovered {
  background-color: white;
}

.last-price {
  margin-right: 10px;
}
</style>
