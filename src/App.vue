<template>
  <div id="app"
    :class="typeof weather.main != 'undefined' && weather.main.temp > 16 ? 'warm': ''"
    v-cloak
    >
    <div class="gradient">
      <main>
        <div class="search-box">
            <input
              type="text"
              class="search-bar" 
              placeholder="Search..."
              v-model="query"
              @keypress="fetchWeather"
              >
            <small v-if="error">{{error}}</small>
        </div>

        <div class="weather-wrapper" v-if="typeof weather.main != 'undefined'">
          <h1 class="weather-today">Today in {{weather.name}}:</h1>
          <!-- <div class="location-box">
            <div class="location">{{weather.name}}, {{weather.sys.country}}</div>
          </div> -->
          <div class="weather-box">
            <div class="temp">{{Math.round(weather.main.temp)}}°C</div>
            <p class="weather">{{weather.weather[0].main}}</p>
            <button class="viewmore" v-if="!view" @click="largeView(true)">View more</button>
            <button class="viewmore" v-else @click="largeView(false)">View less</button>
            <div class="chart" v-if="view">
              <h3>Rain Forecast</h3>
              <chart class="chart" :options="chart"></chart>
            </div>
          </div>
          <div class="weather-week">
            <h2 class="weather-chart">Weekly chart:</h2>
            <div class="week">
              <ul>
                <li
                v-for="(item, index) in chart_temp"
                :key="item.id"
                >{{dateBuilder(index)}} Temp: {{Math.round(item.temp)}}°C Weather: {{item.weather}}</li>
              </ul>
            </div>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
import dateformat from 'dateformat';

export default {
  name: 'App',
  data() {
    return {
      api_key: '4a52d544d254afc86a858c7d743728f9',
      query: '',
      api_base: 'https://api.openweathermap.org/data/2.5/',
      weather: {},
      weather_chart: {},
      chart_temp: [],
      chart_rain: [],
      error: '',
      view: false,
      chart: {
        xAxis: {},
        yAxis: {type: 'value',},
        series: [{type: "line",}],
        title: {
          text: "Monthly Stock Prices",
          x: "center",
          textStyle: {
            fontSize: 24
          }
        },
      }
    };
  },
  methods: {
    fetchWeather(e) {
      if (e.key == 'Enter'){
        try {
          this.chart_temp = [];
          this.weather_chart = {};
          fetch(`${this.api_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
            .then((res) => {
              return res.json();
            }).then(this.setResults)
            .catch((err) => {
              console.log(err);
              this.error = 'You passed wrong name';
              throw new Error(err);
            });
          fetch(`${this.api_base}forecast?q=${this.query}&units=metric&APPID=${this.api_key}`)
            .then((res) => {
              return res.json();
            }).then((res) => {
              this.weather_chart = res;
              var object = {};
              var sum = 0;
              var index = 1
              for(var i=0;i<res.list.length;i++){
                if (index == 8) {
                  index = 1;
                  sum = sum / 8;
                  object.id = res.list[i].dt;
                  object.temp = sum;
                  object.weather = res.list[i].weather[0].main;
                  this.chart_temp.push(object);
                  sum = 0;
                  object = {};
                }
                else {
                  sum += res.list[i].main.temp;
                  index++;
                }
              }
            });
        } catch (error) {
          console.log(error);
          this.error = 'You passed wrong name';
          return;
        }
      }
    },
    setResults(result) {
      this.weather = result;
      this.query = '';
    },
    dateBuilder(index) {
      var d = new Date();
      // let months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
      // let days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

      // let day = days[d.getDay()+index];
      d.setDate(d.getDate()+index);
      var date = dateformat(d, 'ddd mmm yyyy')
      // let month = months[d.getMonth()+index];
      // let year = d.getFullYear()+index;

      return `${date}`;
    },
    largeView() {
      this.chart_rain = [];
      var index = 8;
      var object = {};
      var arrayRain = [];
      var arrayDays = [];
      for(var i=0;i<index;i++){
        var d = new Date(this.weather_chart.list[i].dt*1000);
        d.setDate(d.getDate());
        d = dateformat(d, 'HH : MM');
        console.log(d);
        arrayDays.push(d);
        if (this.weather_chart.list[i].rain){
          arrayRain.push(Object.values(this.weather_chart.list[i].rain)[0]);
        } else { arrayRain.push(0); }
        this.chart_rain.push(object);
        object = {};
      }
      this.chart.series[0].data = arrayRain;
      this.chart.xAxis.data = arrayDays;
      this.view = !this.view;
    },
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;700;900&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'Poppins', sans-serif;
}
.weather-chart{
  font-weight: 500;
  font-size: 30px;
}
small {
  color: rgb(197, 51, 51);
  font-weight: 500;
}
#app {
  background-image: linear-gradient(to top, #89f7fe 0%, #66a6ff 100%);  background-size: cover;
  background-position: bottom;
  -webkit-transition: all 0.4s ease;
  -moz-transition: all 0.4s ease;
  -o-transition: all 0.4s ease;
  transition: 0.4s;
}
/* .gradient{
  background-image: linear-gradient(to bottom, rgba(0,0,0,0.25), rgba(0,0,0,0.75));
} */

#app.warm {
  background-image: linear-gradient(to top, #fa709a 0%, #fee140 100%);
}
.weather-today{
  font-size: 36px;
  font-weight: 500;
}
main {
  min-height: 100vh;
  padding: 30px;
  max-width: 800px;
  margin: 0 auto;
}
.no-padding{
  padding: 0!important;
  margin: 0!important;
}
.search-box {
  width: 100%;
  margin-bottom: 20px;
  max-width: 500px;
  margin: 0 auto;
}
.search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #000;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;
  box-shadow: 0 0 8px rgba(0,0,0,0.25);

  background-color: rgba(0, 0, 0, 0.35);

  border-radius: 0 16px 16px 16px;
  transition: 0.4s;
}
.search-bar:focus{
  box-shadow: 0 0 16px rgba(0,0,0,0.25);
  background-color: rgba(0, 0, 0, 0.30);
  border-radius: 16px 16px 16px 0;
}
.weather-wrapper{
    margin-top: 30px;
}
.location {
  color: white;
  font-size: 40px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0,0,0,0.45);
}
.date {
  color: white;
  font-size: 25px;
  font-weight: 400;
  text-shadow: 1px 3px rgba(0,0,0,0.45);
  text-align: center;
  font-style: italic;
  text-align: center;
}

.weather-box {
  text-align: center;
  background-color: rgba(0, 0, 0, 0.25);
  box-shadow: 3px 6px rgba(0,0,0,0.25);
  border-radius: 16px;
  margin: 30px auto;
  padding-bottom: 0;
  /* max-width: 50%; */
  max-height: 25%;
  width: auto;
  transition: all 0.5s ease-in;
}

.temp {
  display: inline-block;
  /* padding: 10px 25px; */
  font-size: 110px;
  font-weight: 900;
  color: #ffffff;
  text-shadow: 3px 6px rgba(0,0,0,0.25);
  /* margin: 1rem 0; */
  z-index: 900;
}

.weather {
  color: white;
  font-size: 48px;
  font-weight: 700;
  text-shadow: 3px 6px rgba(0,0,0,0.25);
  text-align: center;
  padding: 0;
  margin: 0;
  z-index: 900;
}

.largeView{
  background-color: #18181E!important;
  position: relative;
  box-shadow: none;
  outline: none;
  height: 110vh!important;
  width: 100vw!important;
  text-align: center;
  top: -30px;
  left: 0;
  border-radius: 0;
  padding: 50%;
  margin: 0!important;
}
.viewmore{
  background: none;
  color: white;
  outline: none;
  border: none;
  border-radius: 15px;
  padding: 7px;
  margin: 0 0 0 auto;
  margin-left: 80%;
  font-size: 16px;
}
h3{
  color: white;
  text-shadow: 2px 3px rgba(0,0,0,0.25);
  font-size: 23px;
  margin: 0;
  padding: 0;
}
.chart{
  width: 100%;
  padding: 0;
  margin: 0 auto;
}
ul {
  list-style: none;
}
li{
  color: #ffffff;
  font-size: 22px;
  background-color: rgba(0, 0, 0, 0.25);
  border-radius: 20px;
  padding: 0.6rem;
  margin: 7px auto;
};
</style>
