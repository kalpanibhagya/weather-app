<template>
  <header class="forecast-header">
    <h1>Weather Forecast</h1>
    <div class="input-bar">
      <input v-model="city" placeholder="Location (City,State,Country)" />
      <button @click="getForecast">Get Forecast</button>
    </div>
  </header>
  <div class="weather-app">
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
      </div>
      <div>
        <ForecastTabs
          v-if="forecast.length > 0"
          :forecast="forecast"  
        />
      </div>
    </div>    
  </div>
</template>


<script>
import WeatherWidget from "./components/WeatherWidget.vue";
import ForecastTabs from "./components/ForecastTabs.vue";
const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY;

export default {
  components: { WeatherWidget, ForecastTabs },
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
          this.error = "Location not found. Please check the city, state, country format. If state is not applicable, use City,Country fprmat.";
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

.forecast-header {
  background: linear-gradient(to right, #114b96, #0b2f63);
  color: white;
  padding: 20px;
  text-align: center;
}

.forecast-header h1 {
  margin: 0 0 10px 0;
  font-size: 2rem;
}

.input-bar {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.input-bar input {
  padding: 8px 12px;
  border-radius: 5px;
  border: none;
  font-size: 1rem;
}

.input-bar button {
  padding: 8px 16px;
  border-radius: 5px;
  border: none;
  background-color: #ffcc00;
  cursor: pointer;
  font-weight: bold;
  color: #0b2f63;
  transition: background-color 0.3s;
}

.input-bar button:hover {
  background-color: #ffaa00;
}
</style>