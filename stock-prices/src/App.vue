<template>
  <div class="main">
    <h1>Монитор акций</h1>
    <div class="chart-holder">
      <div class="inputs-holder">
        <div class="dropdown">
          <label for="stockSelect">Выберите акцию:</label>
          <select
            id="stockSelect"
            v-model="selectedStock"
            @change="fetchStockPriceData"
          >
            <option value="AAPL">Apple Inc.</option>
            <option value="MSFT">Microsoft Corporation</option>
            <option value="IBM">IBM Electronics</option>
          </select>
        </div>
        <div class="date-inps">
          <input
            class=""
            type="date"
            v-model="startDate"
            @change="FilterData"
          />
          <input type="date" v-model="endDate" @change="FilterData" />
        </div>
      </div>
      <Chart :stockPriceData="filteredData" />
      <div class="analitics-txt">Цена, $</div>
      <div class="analitics-block">
        <div>Минимальная : {{ this.minValue }}</div>
        <div>Максимальная : {{ this.maxValue }}</div>
        <div>Средняя : {{ this.averageValue }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import Chart from "@/components/Chart.vue";
import axios from "axios";

export default {
  data() {
    return {
      startDate: "",
      endDate: "",
      stockPriceData: [],
      filteredData: [],
      selectedStock: "IBM",
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
      let req =
        "https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol=" +
        this.selectedStock +
        "&apikey=6S3XWKYVUZIUJZEF";
      //6S3XWKYVUZIUJZEF
      try {
        const response = await axios.get(req);
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
        this.FilterData();
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
  mounted() {
    const today = new Date();
    this.endDate = today.toISOString().substr(0, 10);

    const oneMonthAgo = new Date(
      today.getFullYear(),
      today.getMonth() - 1,
      today.getDate()
    );
    this.startDate = oneMonthAgo.toISOString().substr(0, 10);

    this.fetchStockPriceData();
    // this.FilterData();
  },
};
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Fira sans", sans-serif;
}

h1 {
  margin-top: 2vw;
  margin-bottom: 2vw;
}

.main {
  margin: auto;
  width: 80%;
}

.analitics-txt {
  margin-bottom: 1vw;
  margin-top: 2vw;
  font-weight: bold;
}

.chart-holder {
  border-radius: 2vw;
  box-shadow: 2px 4px 20.6px 0px rgba(0, 0, 0, 0.15);
  padding: 3vw;
}

.dropdown {
  display: flex;
  gap: 1vw;
}

.date-inps {
  display: flex;
  gap: 1vw;
}

.inputs-holder {
  display: flex;
  justify-content: space-between;
}

.analitics-block {
  width: 80%;
  display: flex;
  justify-content: space-between;
}
</style>
