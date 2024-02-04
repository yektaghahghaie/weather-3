<script>
import axios from "axios";
import Chart from "chart.js/auto";
import moment from "moment";

export default {
  data() {
    return {
      weatherData: null,
      forecastData: [],
      recentCities: [],
      selectedDay: null, 
      showChart: false,
      city: "",
      day:"",
      unit: "°C",
      apiKey: "1ca7258aa3d87284c748defdf36706b5",
      weatherData: null,
      selectedDayForecast: null,
      selectedHourlyForecast: null,
      hourlyChart: null,
      weeklyForecasts: [],
      weatherIcon: "",
      temperature: null,
      description: null,
      iconUrl: null,
      date: null,
      time: null,
      name: null,
      country: null,
      forecast: [],
      dayOfWeek:"",
      loading: true,
    };
  },
  mounted() {
    this.loadRecentCities();
  },
  methods: {
    searchWeather() {
      axios
        .get(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${this.apiKey}`
        )
        .then((response) => {
          this.weatherData = response.data;
          const d = new Date();
          this.date =
            d.getDate() +
            "-" +
            this.monthNames[d.getMonth()] +
            "-" +
            d.getFullYear();
          this.time =
            d.getHours() + ":" + d.getMinutes() + ":" + d.getSeconds();
          this.addToRecentCities(response.data);
          //this.today = this.getDayOfWeek(d);
        })
        .catch((error) => {
          console.error(error);
        });

      axios
        .get(
          `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&appid=${this.apiKey}&units=metric`
        )
        .then((response) => {
          this.forecastData = response.data.list.slice(0, 5).map((day) => {
            const date = new Date(day.dt * 1000);
            day.dayOfWeek = this.getDayOfWeek(date);
            return day;
          });
        })
        .catch((error) => {
          console.error(error);
        });
    },
    getWeatherForRecentCity(cityId) {
      const recentCity = this.recentCities.find((city) => city.id === cityId);
      this.weatherData = recentCity;
      this.city = recentCity.name;
      this.searchWeather();
    },
    addToRecentCities(cityData) {
      if (!this.recentCities.some((city) => city.id === cityData.id)) {
        this.recentCities.unshift(cityData);
        if (this.recentCities.length > 3) {
          this.recentCities.pop();
        }
        localStorage.setItem("recentCities", JSON.stringify(this.recentCities));
      }
    },
    saveRecentCity(city, country, temperature) {
      let recentCities = JSON.parse(localStorage.getItem("recentCities")) || [];
      recentCities.unshift({ city, country, temperature });
      recentCities = recentCities.slice(0, 3); // حداکثر 3 شهر ذخیره شده را نگه دارید
      localStorage.setItem("recentCities", JSON.stringify(recentCities));
      this.recentCities = recentCities;
    },
    loadRecentCities() {
      const recentCities = JSON.parse(localStorage.getItem("recentCities"));
      if (recentCities) {
        this.recentCities = recentCities;
      }
    },
    convertTemperature(temp) {
      if (this.unit === "°C") {
        return temp;
      } else {
        return (temp * 9) / 5 + 32;
      }
    },
    toggleUnit() {
      if (this.unit === "°C") {
        this.unit = "°F";
      } else {
        this.unit = "°C";
      }
    },
    getWeatherIconURL(iconCode) {
      return `https://api.openweathermap.org/img/w/${iconCode}.png`;
    },
    convertUnixTime(unixTime) {
      return new Date(unixTime * 1000);
    },
    selectDayForecast(day) {
      this.selectedDayForecast = day;
      this.selectedDayTemperature = day.main.temp ;
    },
    async selectDayForecast(day) {
      this.selectedDayForecast = day;
      const response = await axios.get(
        `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&appid=${this.apiKey}&units=metric`
      );
      const forecastData = response.data.list;
      const selectedDayData = forecastData.filter((data) => {
        return (
          new Date(data.dt * 1000).toDateString() ===
          new Date(day.dt * 1000).toDateString()
        );
      });
      const labels = selectedDayData.map(
        (data) => new Date(data.dt * 1000).getHours() + "AM"
      );
      const temperatures = selectedDayData.map((data) => data.main.temp);
      const ctx = document.getElementById("hourlyChart");
      if (this.hourlyChart) {
        this.hourlyChart.destroy();
      }
      this.hourlyChart = new Chart(ctx, {
        type: "bar",
        data: {
          labels: labels,
          datasets: [
            {
              label: "Hourly Temperature (°C)",
              data: temperatures,
              backgroundColor: "rgba(75, 192, 192, 0.2)",
              borderColor: "rgba(75, 192, 192, 1)",
              borderWidth: 1,
            },
          ],
        },

        options: {
          scales: {
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: "temprature(c)",
              },
            },
            x: {
              title: {
                display: true,
                text: "time",
              },
            },
          },
        },
      });
    },

    getDayOfWeek(date) {
      const daysOfWeek = ["s", "d", "y", "sh", "g", "p", "ch"];
      const d = new Date(date * 1000);
      return daysOfWeek[d.getDay()];
    },
  },
  filters: {
    formatDate(date) {
      return date.toLocaleDateString("fa-IR");
    },
  },
};
</script>
<template>
  <div>
    <input
      type="text"
      v-model="city"
      @keyup.enter="searchWeather"
      placeholder="نام شهر را وارد کنید"
    />
    <button @click="toggleUnit">Toggle Unit</button>
    <div v-if="weatherData">
      <h2>{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
      <h3>امروز ({{ weatherData.day }})</h3>
      <p>دما: {{ convertTemperature(weatherData.main.temp) }} {{ unit }}</p>
      <img
        :src="getWeatherIconURL(weatherData.weather[0].icon)"
        :alt="weatherData.weather[0].description"
      />
      <p>
        طلوع خورشید: {{ convertUnixTime(weatherData.sys.sunrise) | formatDate }}
      </p>
      <p>
        غروب خورشید: {{ convertUnixTime(weatherData.sys.sunset) | formatDate }}
      </p>
      <p>{{ time }}</p>
      <p>فشار هوا: {{ weatherData.main.pressure }} هکتوپاسکال</p>
      <p>
        Feels Like: {{ convertTemperature(weatherData.main.feels_like) }}
        {{ unit }}
      </p>
    </div>
    <div>
      <h3>شهرهای اخیر</h3>
      <ul>
        <li
          v-for="recentCity in recentCities"
          :key="recentCity.id"
          @click="getWeatherForRecentCity(recentCity.id)"
        >
          {{ recentCity.name }}, -{{ weatherData.sys.country }}
          {{ recentCity.weather[0].description }}
          <img
            :src="getWeatherIconURL(recentCity.weather[0].icon)"
            :alt="recentCity.weather[0].description"
          />
        </li>
      </ul>
    </div>
    <div>
      <h3>روزهای هفته</h3>
      <ul v-if="forecastData.length > 0">
        <li
          v-for="day in forecastData"
          :key="day.dt"
          @click="selectDayForecast(day)"
        >
          {{ day.dayOfWeek }} - دما: {{ convertTemperature(day.main.temp) }}
          {{ unit }}
          <img
            :src="getWeatherIconURL(day.weather[0].icon)"
            :alt="day.weather[0].description"
          />
        </li>
      </ul>
    </div>
    <div v-if="selectedDayForecast">
      <h3>Hourly Temperature Forecast</h3>
      <canvas id="hourlyChart" width="400" height="200"></canvas>
    </div>
    <div v-if="selectedDayForecast">
      <h3>overview</h3>
      <p>
        Date: {{ new Date(selectedDayForecast.dt * 1000).toDateString() }}'s
        overview
      </p>
      <p>
        Temperature: {{ convertTemperature(selectedDayForecast.main.temp)
        }}{{ unit }}
      </p>
      <p class="overview">
        Max Temperature:
        {{ convertTemperature(selectedDayForecast.main.temp_max) }}{{ unit }}
      </p>
      <p class="overview">
        Min Temperature:
        {{ convertTemperature(selectedDayForecast.main.temp_min) }}{{ unit }}
      </p>
      <p class="overview">
        Wind Speed: {{ selectedDayForecast.wind.speed }} m/s
      </p>
      <p class="overview">Humidity: {{ selectedDayForecast.main.humidity }}%</p>
      <p>جهت باد: {{ selectedDayForecast.wind.deg }} درجه</p>
    </div>
  </div>
</template>

<style scoped>
/** {
  padding: 0;
  margin: auto;
  box-sizing: border-box;
  background-color: #111015;
}
input {
  background-color: #1e1e1e;
  width: 300px;
  margin-top: 10px;
  margin-left: 100px;
  border-radius: 4px;
}
.toggle {
  margin-left: 350px;
  margin-top: 10px;
}
.header {
  display: flex;
}
.serch {
  position: absolute;
  top: 10px;
  left: 90px;
}
.codr {
  position: relative;
}
.recentcity {
  background-color: #202022b8;
  border-radius: 10px;
  width: 230px;
  he

ight: 100px;
  list-style-type: none;
}
.overview {
  background-color: #202022b8;
  border-radius: 10px;
  width: 100px;
  height: 100px;
}
.footer {
  display: flex;
}*/
</style>





































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































