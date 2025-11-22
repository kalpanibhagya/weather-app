<template>
  <div v-if="weather" class="widget">
    <h2 class="location">
      {{ weather.name }}, {{ weather.sys.country }}
    </h2>

    <div class="current-section">
      <div class="left">
        <img
          class="icon"
          :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@4x.png`"
          alt="Weather icon"
        />

        <div class="temp">
          {{ toC(weather.main.temp) }}°C
        </div>

        <div class="desc">
          {{ weather.weather[0].description }}
        </div>

        <div class="time">
          Updated: {{ formattedTime }}
        </div>
      </div>

      <div class="right">
        <div class="row"><span>Wind</span><span>{{ (weather.wind.speed * 3.6).toFixed(0) }} km/h</span></div>
        <div class="row"><span>Humidity</span><span>{{ weather.main.humidity }}%</span></div>
        <div class="row"><span>Feels like</span><span>{{ toC(weather.main.feels_like) }}°C</span></div>
        <div class="row"><span>Visibility</span><span>{{ (weather.visibility / 1000).toFixed(1) }} km</span></div>
        <div class="row"><span>Pressure</span><span>{{ weather.main.pressure }} hPa</span></div>
        <div class="row"><span>Dew point</span><span>{{ dewPoint }}°</span></div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  props: {
    lat: { type: Number, required: true },
    lon: { type: Number, required: true },
    apiKey: { type: String, required: true }
  },

  data() {
    return {
      weather: null,
      loading: false,
      error: ""
    };
  },

  computed: {
    formattedTime() {
      const t = new Date((this.weather.dt + this.weather.timezone) * 1000);
      return t.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    },

    dewPoint() {
      const T = this.toC(this.weather.main.temp);
      const RH = this.weather.main.humidity;

      const a = 17.27, b = 237.7;
      const alpha = (a * T) / (b + T) + Math.log(RH / 100);
      return Math.round((b * alpha) / (a - alpha));
    }
  },

  methods: {
    toC(kelvin) {
      return Math.round(kelvin - 273.15);
    },

    async fetchWeather() {
      try {
        this.loading = true;
        this.error = "";
        console.log("Fetching weather for", this.lat, this.lon);
        const url = `https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}`;

        const res = await fetch(url);
        if (!res.ok) throw new Error("Failed to fetch weather");

        this.weather = await res.json();
      } catch (e) {
        this.error = e.message;
      } finally {
        this.loading = false;
      }
    }
  },

  watch: {
    lat() { this.fetchWeather(); },
    lon() { this.fetchWeather(); }
  },

  mounted() {
    this.fetchWeather();
    setInterval(this.fetchWeather, 10 * 60 * 1000); // auto-refresh every 10 minutes
  }
};
</script>

<style scoped>
.widget {
  /* width: 500px; */
  background: linear-gradient(to bottom right, #75a1db, #557fbb);
  color: white;
  padding: 40px;
  border-radius: 20px;
  /* background: rgba(100, 150, 255, 0.15);  
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.25);
  box-shadow: 0 8px 25px rgba(0, 0, 100, 0.25);   */
}

.location {
  font-size: 30px;
  margin-bottom: 10px;
}

.current-section {
  display: flex;
  justify-content: space-between;
}

.left {
  text-align: center;
}


.temp {
  font-size: 45px;
  font-weight: bold;
}

.desc {
  text-transform: capitalize;
  margin-top: 5px;
  font-size: 18px;
}

.time {
  margin-top: 5px;
  opacity: 0.6;
  font-size: 14px;
}

.right {
  font-size: 18px;
  width: 200px;
}

.row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}
</style>
