<template>
  <div>
    <div id="tab" class="container">
      <ul>
        <li @click="activeTab = '1'" :class="[activeTab === '1' ? 'active' : '']"> Today's forecast
        </li>
        <li @click="activeTab = '2'" :class="[activeTab === '2' ? 'active' : '']"> Five days forecast
        </li>
      </ul>
      <div class="tabs-content">
        <div class="content-one" v-if="activeTab === '1'">
          <div 
            class="card"
            v-for="data in weatherData" 
            :key="data.index"
            :class="{'favourite-item': data.favourite}"
          > 
            <div class="btn-block">
              <input class="regular-btn" type="button" value="Remove from favourite" @click="removeFromFavourite(data.id, data.name)" />
            </div>
            
            <span class="first-block">
              <p class="city">{{ data.name }}, {{ data.country }}</p>
              <p>{{ Math.round(data.temp) }}°</p>
            </span>
            <span class="second-block">
              <p class="weather">{{ data.weather }}</p>
              <p>Feels like: {{ Math.round(data.feelsLike) }}°</p>
            </span>
            <p>Wind: 
              <span>
                <img :style="`transform:rotate(${data.windDeg}deg)`" class="wind-arrow" src="../assets/arrow.png" alt="arrow">
              </span> {{ data.windSpeed }} meters per second</p>
            <div>
              
            </div>
            <p>Humidity: {{ data.humidity }}%</p>
            <p>Pressure: {{ data.pressure }}hPa</p>
          </div>
        </div>
        <div class="content-two" v-if="activeTab === '2'">
          <div 
            class="card"
            v-for="one in fiveForecast" 
            :key="one.index"
            :class="{'favourite-item': one.favourite}"
          > 
            <div class="btn-block">
              <input class="regular-btn" type="button" value="Remove from favourite" @click="removeFromFavourite(one.id, one.name)" />
            </div>
            
            <div class="first-block">
              <p class="city">{{ one.name }}, {{ one.country }}</p>
              
            </div>
            <div v-for="(forecast, index) in one.forecastList" :key="index" class="forecast-temp"> 
              <p>
                {{ new Date(forecast.dt_txt).getDate() }}.{{ new Date(forecast.dt_txt).getMonth() + 1 }}:</p>
              <p>
                {{ Math.round(forecast.main.temp) }}, {{ forecast.weather[0].description }}
              </p>
            </div>
            
          </div>
          <p></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'FavouritePage',
  data: () => ({
    activeTab: '1',
    city: '',
    locationKey: '691a16990bb9a70332e47bcd8fedfcf6',
    weatherKey: 'b3159520889c26993eea38da6ac0ea9d',
    index: '',
    weatherData: [{}],
    fiveForecast:[{}]
  }),
  methods: {

    removeFromFavourite(id, name) {
      alert(`${name} was removed from favourite!`)
      localStorage.removeItem(id, name)
    },
  },
  async created() {
    
    this.weatherData.splice(0, 1)
    this.fiveForecast.splice(0, 1)
    let localStorageKeys = []
    for(let i = 0; i < localStorage.length; i++) {
      localStorageKeys.push(localStorage.getItem(localStorage.key(i)))
    }

    for(let i = 0; i < localStorageKeys.length; i++) {
      let city = localStorageKeys[i]
      let lat
      let lon
      ++this.index
      
      await(await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${city}&appid=${this.locationKey}`)
        .then((response) => response.json())
          .then((data) => {lat = data[0].lat, lon = data[0].lon}))

      let weather = await(await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&exclude=minutely,hourly,daily,alerts&appid=${this.weatherKey}`)
        .then((response) => response.json())
          .then((result) => result))  
      this.weatherData.push({
        id: weather.id,
        index: this.index,
        name: weather.name,
        country: weather.sys.country,
        temp: weather.main.temp,
        weather: weather.weather[0].description,
        feelsLike: weather.main.feels_like,
        windSpeed: weather.wind.speed,
        windDeg: weather.wind.deg,
        humidity: weather.main.humidity,
        pressure: weather.main.pressure,
        favourite: true
      })

      let five = await(await fetch(`https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=metric&exclude=minutely,hourly,daily,alerts&appid=${this.weatherKey}`)
        .then((response) => response.json())
          .then((result) => result))
      this.fiveForecast.push({
        id: five.city.id,
        index: this.index,
        name: five.city.name,
        country: five.city.country,
        forecastList: (function() {
          const result = []
          for(let i = 0; i < five.list.length; i+=8) {
            result.push(five.list[i])
          }
          return result
        })(),
        favourite: true
      })
    }
  }
}
</script>

<style scoped>

 /* Tabs styles*/
.container {
  margin: 20px auto;
  width: 1200px;
}

.tabs-choose {
  margin-bottom: 0;
}

li {
  display: inline-block;
  padding: 10px 20px;
  background-color: #eee;
  cursor: pointer;
  opacity: 0.3;
}

.active {
  opacity: 1;
  font-weight: bold;
  background-color: lightblue
}

.tabs-content {
  margin: 0 auto;
  padding: 20px;
}


.links-urls {
  position: absolute;
  bottom: 0;
  right: 0;
  padding: 10px;
}

.link {
  padding: 10px 20px;
  display: block;
  background-color: #EEE;
  margin-bottom: 10px;
  text-decoration: none;
  color: #777;
}



 /* Tabs styles ends here*/



  .forecast-temp {
    display: flex;
    justify-content: space-between;
    font-size: 20px;
    font-weight: bold;
    border-bottom: 1px solid lightblue;
  }
  .btn-block {
    text-align: end;
  }

  .card {
    display: inline-block;
    padding: 10px 20px;
    width: 250px;
    margin: 40px;
    border: 1px solid lightblue;
    text-align: left;
    font-size: 14px;
  }

  .favourite-item {
    border: 4px solid gold;
    border-radius: 10px;
  }

  .regular-btn {
    background-color: transparent;
    border: 4px solid gold;
    border-radius: 10px;
    font-weight: bold;
    cursor: pointer;
  }

  .first-block {
    display: flex;
    height: 90px;
    justify-content: space-between;
    font-size: 24px;
    font-weight: bold;
  }

  .second-block {
    display: flex;
    height: 90px;
    justify-content: space-between;    
    text-align: start;
    font-style: italic;
    font-size: 20px;
  }

  .city {
    text-align: start;
  }

  .weather {
    margin-right: 5px;
  }

  .weather::first-letter {
    text-transform: uppercase;
  }

  .wind-arrow {
    transform: rotate(0deg);
    width: 20px;
    vertical-align: middle;
    padding: 0 10px;
  }

</style>