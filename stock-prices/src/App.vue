<template>
  <div>
    <input type="date" v-model="startDate" @change="FilterData" />
    <input type="date" v-model="endDate" @change="FilterData" />
    <button @click="fetchStockPriceData">Fetch Stock Price Data</button>
    <Chart :stockPriceData="filteredData" />
    <div>Minimum : {{ this.minValue }}</div>
    <div>Maximum : {{ this.maxValue }}</div>
    <div>Average : {{ this.averageValue }}</div>
  </div>
</template>

<script>
import Chart from "@/components/Chart.vue";
// import { ref } from "vue";
import axios from "axios";
// import Chart from './Chart.vue';

export default {
  data() {
    return {
      startDate: "",
      endDate: "",
      stockPriceData: [],
      filteredData: [],
      averageValue: 0,
      minValue: 0,
      maxValue: 0,
    };
  },
  components: {
    Chart,
  },
  methods: {
    async fetchStockPriceData() {
      try {
        const response = await axios.get(
          "https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=IBM&apikey=demo"
        );
        var dataObject = response.data["Time Series (Daily)"];
        this.stockPriceData = Object.keys(dataObject)
          .map((date) => {
            return {
              date: date,
              price: dataObject[date]["4. close"],
            };
          })
          .reverse();
        console.log(this.stockPriceData[0]);
        this.filteredData = this.stockPriceData;
      } catch (error) {
        console.error("Error fetching stock price data:", error);
      }
    },
    FilterData() {
      console.log(new Date(this.stockPriceData[0].date));

      this.filteredData = this.stockPriceData.filter((data) => {
        return (
          (!this.startDate ||
            new Date(data.date) >= new Date(this.startDate)) &&
          (!this.endDate || new Date(data.date) <= new Date(this.endDate))
        );
      });

      var prices = this.filteredData.map((data) => data.price);

      this.minValue = Math.min(...prices);
      this.maxValue = Math.max(...prices);
      this.averageValue =
        prices.reduce((acc, curr) => parseFloat(acc) + parseFloat(curr), 0) /
        prices.length;
    },
  },
  computed: {},
  mounted() {},
};
</script>

<style lang="scss"></style>
