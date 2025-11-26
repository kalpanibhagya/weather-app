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
              <img
                :src="`https://openweathermap.org/img/wn/${dailyIcons[day]}@2x.png`"
                alt="icon"
                class="tab-icon"
              />
              <div class="day-name">{{ formatDay(day) }}</div>
              <div class="day-date">{{ day }}</div>
              <div class="temp-range">
                ({{ dailyMinMax[day].min }}° / {{ dailyMinMax[day].max }}°)
              </div>
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
        dailyMinMax() {
          const result = {};
          for (const [day, entries] of Object.entries(this.groupedForecast)) {
              const temps = entries.map(e => e.main.temp - 273.15);
              const min = Math.min(...temps);
              const max = Math.max(...temps);
              result[day] = {
                  min: Math.round(min),
                  max: Math.round(max),
              };
          }
          return result;
      },
      dailyIcons() {
        const icons = {};
        for (const [day, entries] of Object.entries(this.groupedForecast)) {
          const noonEntry = entries.find(e => e.dt_txt.includes("12:00:00"));
          const chosen = noonEntry || entries[0]; // fallback
          icons[day] = chosen.weather[0].icon;
        }
        return icons;
      }
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
        formatDay(dayString) {
          const date = new Date(dayString);
          return date.toLocaleDateString("en-US", { weekday: "short" }); // Tue, Wed
        }
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
  background: linear-gradient(to bottom right, #4A90E2, #c0c3c8);
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
  gap: 14px;
  margin-bottom: 20px;
  overflow-x: auto;
  padding-bottom: 6px;
}

.tabs button {
  position: relative;
  width: 160px;
  height: 160px;
  padding: 12px;
  border: none;
  border-radius: 16px;
  cursor: pointer;
  background: #4A90E2;
  color: white;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-shadow: 0 4px 10px rgba(0,0,0,0.15);
  transition: transform 0.15s ease, background 0.15s ease;
}

.tabs button:hover {
  transform: translateY(-3px);
}

.tabs button.active {
  background: #114b96;
}

.tab-icon {
  width: 55px;
  height: 55px;
  margin: 0 auto 4px auto;
}

.day-name {
  font-size: 1.1rem;
  font-weight: 600;
}

.day-date {
  font-size: 0.9rem;
  opacity: 0.9;
}

.temp-range {
  font-size: 0.95rem;
  font-weight: 500;
  margin-top: 6px;
}

.today-card {
  margin-top: 20px;
  padding: 40px;
  border-radius: 20px;
  background: linear-gradient(to bottom right, #4A90E2, #c0c3c8);
  color: white;
  box-shadow: 0 4px 16px rgba(0,0,0,0.2);
}

.today-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 0;
  border-bottom: 1px solid rgba(255,255,255,0.15);
}

.today-item:last-child {
  border-bottom: none;
}

.time {
  font-size: 1.2rem;
  font-weight: 600;
  width: 80px;
}

.temp {
  font-size: 1.4rem;
  font-weight: 600;
  width: 80px;
}

.desc {
  font-size: 1.1rem;
  text-transform: capitalize;
  width: 160px;
}

.details {
  text-align: right;
  line-height: 1.6;
  opacity: 0.9;
  font-size: 1.05rem;
}


</style>