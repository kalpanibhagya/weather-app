<template>
  <div>
    <line-chart :chart-data="chartData" :chart-options="chartOptions" />
  </div>
</template>

<script>
import { defineComponent, computed } from "vue";
import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement } from "chart.js";
import { Line } from "vue-chartjs";

// Register chart.js components
ChartJS.register(Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement);

export default defineComponent({
  name: "ForecastChart",
  components: {
    LineChart: Line,
  },
  props: {
    forecast: { type: Array, required: true },
    selectedDay: { type: String, required: true },
  },
  computed: {
    // Prepare grouped data
    groupedForecast() {
      const grouped = {};
      this.forecast.forEach((item) => {
        const day = item.dt_txt.split(" ")[0];
        if (!grouped[day]) grouped[day] = [];
        grouped[day].push(item);
      });
      return grouped;
    },
    chartData() {
      if (!this.selectedDay || !this.groupedForecast[this.selectedDay]) return null;

      const labels = this.groupedForecast[this.selectedDay].map(item => item.dt_txt.split(" ")[1]); // time
      const temps = this.groupedForecast[this.selectedDay].map(item => Math.round(item.main.temp - 273.15));

      return {
        labels,
        datasets: [
          {
            label: "Temperature (°C)",
            data: temps,
            fill: false,
            borderColor: "#114b96",
            backgroundColor: "#114b96",
            tension: 0.3
          }
        ]
      };
    },
    chartOptions() {
      return {
        responsive: true,
        plugins: {
          legend: { display: true },
          tooltip: { enabled: true }
        },
        scales: {
          y: { beginAtZero: false }
        }
      };
    }
  }
});
</script>

<style scoped>
</style>
