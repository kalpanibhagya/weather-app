<template>
  <div class="weather-app">
    <h1>Weather Forecast</h1>

    <div class="input-bar">
      <input v-model="city" placeholder="Location (City,State,Country)" />
      <button @click="getForecast">Get Forecast</button>
    </div>
    <div v-if="loading">Loading...</div>
    <div v-if="error" style="color:red">{{ error }}</div>

    <div class="grid">
      <div>
        <WeatherWidget
          v-if="lat && lon"
          :lat="lat"
          :lon="lon"
          :apiKey="apiKey"
        />
        <div v-if="todayData.length > 0" class="today-card">
          <h2>Today's Weather</h2>

          <div 
              v-for="(item, index) in todayData" 
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
      <!-- <div>
        <div v-if="forecast.length > 0" class="forecast-scroll">
          <h2>Forecast for next 5 days</h2>
          <br></br>
          <ul>
            <li v-for="(item, index) in forecast" :key="index">
              {{ item.dt_txt }} - Temp: {{ Math.round(item.main.temp - 273.15) }}°C - {{ item.weather[0].description }}
            </li>
          </ul>
        </div>
      </div> -->
      <div>
        <div v-if="forecast.length > 0">
          <h2>Forecast for next 5 days</h2>

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

          <!-- Forecast List for selected day -->
          <ul class="forecast-scroll">
            <li v-for="(item, index) in groupedForecast[selectedDay]" :key="index">
              {{ item.dt_txt }} - Temp: {{ Math.round(item.main.temp - 273.15) }}°C - {{ item.weather[0].description }}
            </li>
          </ul>
        </div>
        <!-- <ForecastChart
          v-if="selectedDay"
          :forecast="forecast"  
          :selectedDay="selectedDay" /> -->
      </div>
    </div>    
  </div>
</template>


<script>
// import ForecastChart from "./components/ForecastChart.vue";
import WeatherWidget from "./components/WeatherWidget.vue";
const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY;

export default {
  components: { WeatherWidget },
  data() {
    return {
      lat: "",
      lon: "",
      forecast: [],
      loading: false,
      error: "",
      apiKey: apiKey,
      dailyData: {},
      city: "",
      currentWeather: null,
      selectedDay: null,
    };
  },

  computed: {
    todayData() {
      if (!this.forecast.length) return [];

      const today = new Date().toISOString().split("T")[0];

      return this.forecast.filter(item => item.dt_txt.startsWith(today));
    },
    groupedForecast() {
      return this.groupByDay(this.forecast);
    },
    days() {
      return Object.keys(this.groupedForecast);
    },
  },

  methods: {
    async getForecast() {
      this.loading = true;
      this.error = "";
      if(this.city.length == 0){
        this.error = "Please enter a city with the format City,State,Country. If state is not applicable, use City,Country fprmat.";
        this.loading = false;
        return;
      }
      const cityParts = this.city.split(',');
      const cityName = cityParts[0].trim();
      const stateCode = cityParts.length == 3 ? cityParts[1].trim() : '';
      const countryCode = cityParts.length == 3 ? cityParts[2].trim() : cityParts[1].trim();
      const geoUrl = `http://api.openweathermap.org/geo/1.0/direct?q=${cityName}${stateCode ? ',' + stateCode : ''},${countryCode}&limit=1&appid=${this.apiKey}`;
      try {
        const geoRes = await fetch(geoUrl);
        if (!geoRes.ok) throw new Error("Invalid Geocoding API response");

        const geoData = await geoRes.json();
        if(geoData.length == 0){
          this.error = "Location not found. Please check the city, state, country format.";
          this.loading = false;
          return;
        }
        this.lat = geoData[0].lat;
        this.lon = geoData[0].lon;
      } catch (err) {
        this.error = "Error fetching geocoding data: " + err.message;
        this.loading = false;
        return;
      }

      const url = `https://api.openweathermap.org/data/2.5/forecast?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}`;

      try {
        const res = await fetch(url);
        if (!res.ok) throw new Error("Invalid API response");

        const data = await res.json();
        this.forecast = data.list;

        // group by days
        this.dailyData = this.groupByDay(data.list);
        console.log(this.dailyData);
      } catch (err) {
        this.error = err.message;
      } finally {
        this.loading = false;
      }
    },

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
};
</script>


<style scoped>
.weather-app {
  /* max-width: 500px; */
  margin: 50px;
}
input {
  margin: 4px;
  padding: 6px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
button {
  padding: 6px 12px;
  border-radius: 4px;
  background-color: #4a90e2;
  color: white;
  border: none;
  cursor: pointer;
}
ul {
  padding-left: 0;
  list-style: none;
}
li {
  padding: 4px 0;
}



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
  max-height: 600px; 
  overflow-y: auto;
  padding-right: 8px;
  border: 1px solid #ddd;
  border-radius: 6px;
}

input-bar {
  margin-bottom: 50px;
}

.grid {
  display: grid;
  grid-template-columns: 1fr 2fr; 
  gap: 20px;
}

@media (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr;
  }
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
.forecast-scroll {
  max-height: 300px;
  overflow-y: auto;
}


</style>