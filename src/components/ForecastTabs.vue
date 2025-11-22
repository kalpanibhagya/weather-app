<template>
    <div v-if="forecast.length > 0">
          <!-- Tabs -->
          <div class="tabs">
            <button
              v-for="(day, index) in days"
              :key="index"
              :class="{ active: selectedDay === day }"
              @click="selectedDay = day"
            >
              {{ day }}
            </button>
          </div>
          <div class="today-card">
          <div 
              v-for="(item, index) in groupedForecast[selectedDay]" 
              :key="index" 
              class="today-item"
            >
            <div class="time">{{ item.dt_txt.split(' ')[1].slice(0, 5) }}</div>

            <div class="temp">
              {{ Math.round(item.main.temp - 273.15) }}°C
            </div>

            <div class="desc">
              {{ item.weather[0].description }}
            </div>

            <img
              class="icon"
              :src="`https://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`"
              alt="icon"
            />

            <div class="details">
              <p>Humidity: {{ item.main.humidity }}%</p>
              <p>Clouds: {{ item.clouds.all }}%</p>
              <p>Wind: {{ item.wind.speed }} m/s</p>
            </div>
          </div>
        </div>
        </div>
</template>

<script>
export default { 
    props: {
        forecast: {
            type: Array,
            required: true
        }
    },
     data() {
        return {
            selectedDay: null,
        };
    },
    computed: {
        groupedForecast() {
            return this.groupByDay(this.forecast);
        },
        days() {
            return Object.keys(this.groupedForecast);
        },
    },
    methods: {
        groupByDay(list) {
        const grouped = {};
        list.forEach((item) => {
            const day = item.dt_txt.split(" ")[0];
            if (!grouped[day]) grouped[day] = [];
            grouped[day].push(item);
        });
        return grouped;
        },
    },
  watch: {
    // Set the first day as default when forecast data changes
    groupedForecast: {
      immediate: true,
      handler(newVal) {
        if (this.days.length > 0 && !this.selectedDay) {
          this.selectedDay = this.days[0];
        }
      },
    },
  }
}
</script>

<style scoped>

.today-card {
  margin-top: 20px;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  background: linear-gradient(to bottom right, #1659b1, #0b2f63);
  color:white;
}

.today-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.today-item:last-child {
  border-bottom: none;
}

.time {
  font-weight: bold;
  width: 60px;
}

.temp {
  font-size: 1.3em;
  width: 70px;
}

.desc {
  text-transform: capitalize;
  width: 120px;
}

.details {
  font-size: 1em;
  text-align: right;
}
.forecast-scroll {
  max-height: 1000px; 
  overflow-y: auto;
  padding-right: 8px;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}
.tabs button {
  padding: 8px 16px;
  border: none;
  cursor: pointer;
  background-color: #507597;
  border-radius: 5px;
}
.tabs button.active {
  background-color: #114b96;
  color: rgb(255, 255, 255);
}

</style>