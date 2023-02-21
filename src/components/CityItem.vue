<template>
  <div class="card">
    <h2 class="city-name">{{ this.weatherData.name }}, {{ this.weatherData.country }}</h2>
    
    <div class="weather">
      <div class="main">
        <p class="weather-desc">{{ this.weatherData.weather }}</p>
        <p class="now-is">{{ Math.round(this.weatherData.temp) }}°</p>
      </div>
      <div class="degrees">
        <p>Feels Like: {{ Math.round(this.weatherData.feelsLike) }}°</p>
        <p>Today's minimum: {{ Math.round(this.weatherData.tempMin) }}°</p>
        <p>Today's maximum: {{ Math.round(this.weatherData.tempMax) }}°</p>
      </div>
    </div>
    <div>
      <img src="../assets/sun.jpg" alt="sun">
      <div class="sun">
        <p>{{ this.weatherData.sunrise }}</p>
        <p>{{ this.weatherData.sunset }}</p>
      </div>
    </div>
    <div class="wind">
        <p>Wind: </p>
        <img :style="`transform:rotate(${this.weatherData.windDeg}deg)`" class="wind-arrow" src="../assets/arrow.png" alt="arrow">
        <p>{{ this.weatherData.windSpeed }} meters per second</p>
    </div>
    <!-- <p>Wind Deg: {{ this.weatherData.windDeg }}</p> -->
    <div class="weather">
      <p>Humidity: {{ this.weatherData.humidity }}%</p>
      <p>Pressure: {{ this.weatherData.pressure }} hPa</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CityItem',
  data: () => ({
    locationKey: '691a16990bb9a70332e47bcd8fedfcf6',
    weatherKey: 'b3159520889c26993eea38da6ac0ea9d',
    weatherData: {
    },
  }),
  methods: {
    
  },
  async created() {
    
    let city = this.$route.path
    city = city.replace(/[^a-zа-яё\s]/gi, ' ')
    city = city.replace(/\s+/g, ' ')
    // this.weatherData.splice(0, 1)

    
        let lat
        let lon
        
        await(await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${city}&appid=${this.locationKey}`)
          .then((response) => response.json())
            .then((data) => {lat = data[0].lat, lon = data[0].lon}))

        let weather = await(await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&exclude=minutely,hourly,daily,alerts&appid=${this.weatherKey}`)
          .then((response) => response.json())
            .then((result) => result))  
        this.weatherData = {
          id: weather.id,
          name: weather.name,
          country: weather.sys.country,
          temp: weather.main.temp,
          tempMax: weather.main.temp_max,
          tempMin: weather.main.temp_min,
          weather: weather.weather[0].description,
          sunrise: weather.sys.sunrise,
          sunset: weather.sys.sunset,
          feelsLike: weather.main.feels_like,
          windSpeed: weather.wind.speed,
          windDeg: weather.wind.deg,
          humidity: weather.main.humidity,
          pressure: weather.main.pressure,
        }
        this.weatherData.sunrise = (function(unixTimestamp){
          let a = new Date(unixTimestamp * 1000);
          let hour = a.getHours();
          let min = a.getMinutes();
          let sec = a.getSeconds();
          let time = hour + ':' + min + ':' + sec ;
          return time;
        })(this.weatherData.sunrise)
        this.weatherData.sunset = (function(unixTimestamp){
          let a = new Date(unixTimestamp * 1000);
          let hour = a.getHours();
          let min = a.getMinutes();
          let sec = a.getSeconds();
          let time = hour + ':' + min + ':' + sec ;
          return time;
        })(this.weatherData.sunset)
      
    
  }
}
</script>

<style scoped>
  .card {
    display: inline-block;
    padding: 30px;
    border: 4px solid lightblue;
    border-radius: 20px;
  }

  .weather {
    display: flex;
    justify-content: space-evenly;
  }

  .weather-desc {
    margin-top: 5px;
    margin-bottom: 20px;
    font-size: 25px;
  }

  .weather-desc::first-letter {
    text-transform: uppercase;
  }

  .now-is {
    font-size: 45px;
    font-weight: bold;
    margin: 0;
  }

  .degrees {
    text-align: end;
  }

  .wind-arrow {
    width: 20px;
    vertical-align: middle;
    padding: 0 10px;
  }

  .sun {
    display: flex;
    justify-content: space-evenly;
    font-size: 20px;
    font-weight: bold;
  }

  .wind {
    display: inline-flex;
  }

  .wind img {
    padding: 0 20px;
  }
</style>