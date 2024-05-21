<template>
  <div>
    <canvas ref="chartRef"></canvas>
  </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";
import Chart from "chart.js/auto";

export default {
  props: {
    stockPriceData: {
      type: Array,
      required: false,
    },
  },
  setup(props) {
    const chartRef = ref(null);

    let myChart;

    onMounted(() => {
      if (chartRef.value) {
        myChart = new Chart(chartRef.value, {
          type: "line",
          data: {
            labels: props.stockPriceData.map((data) => data.date),
            datasets: [
              {
                label: "Stock Price",
                data: props.stockPriceData.map((data) => data.price),
                borderColor: "blue",
                borderWidth: 1,
              },
            ],
          },
        });
        const prices = props.stockPriceData.map((data) => data.price);
      }
    });

    watch(
      () => props.stockPriceData,
      (newValue) => {
        if (myChart) {
          myChart.data.labels = newValue.map((data) => data.date);
          myChart.data.datasets[0].data = newValue.map((data) => data.price);

          myChart.update();
        }
      }
    );

    return {
      chartRef,
    };
  },
};
</script>

<style lang="scss" scoped></style>
