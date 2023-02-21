<template>
  <div>
    <form autocomplete="on">
      <div class="autocomplete">
        <input 
          type="text" 
          autocomplete="on" 
          v-model="city" 
          @input='filterCities' 
          placeholder="Enter city" 
          @focus="modal = true" 
          class="autocomplete-item" 
        />
      </div>
      <div v-if='filteredCities && modal && this.city.length > 2'>
        <select size="5" class="variant-select">
          <option 
            v-for="(filteredCity, index) in filteredCities"
            :key="index"
            @click="setCity(filteredCity)"
            v-on:keyup.enter="onEnter(filteredCity)"
            class="variant"
          >
            {{ filteredCity }}
          </option>
        </select>
      </div>
      <input 
        type="button" 
        class="regular-btn"
        value="Get weather"
        @click="getWeather()" 
      />
      <input 
        type="button" 
        class="regular-btn"
        value="Add city"
        :disabled="this.weatherData.length > 4"
        @click="addCity()" 
      />
    </form>

  <div 
    class="card"
    v-for="(data, index) in weatherData" 
    :key="index"
    :class="{'favourite-item': data.favourite}"
  > 
    <div class="btn-block">
      <input :disabled="weatherData.length < 2" class="remove-btn" type="button" value="X" @click="deleteCity(index)" :class="{'favourite-remove-btn': data.favourite}" />
      <input v-if="!data.favourite" class="regular-btn" type="button" value="Add to favourite" @click="addToFavourite(data.id, data.name)" />
      <input v-if="data.favourite" class="regular-btn" type="button" value="Remove from favourite" @click="removeFromFavourite(data.id, data.name)" 
      :class="{'favourite-btn': data.favourite}" />
    </div>
    
    <span class="first-block">
      <p 
        class="city"
        @click="$router.push(`/${data.name}`)"
      >
        {{ data.name }}, {{ data.country }}
      </p>
      <p>{{ Math.round(data.temp) }}°</p>
    </span>
    <span class="second-block">
      <p class="weather">{{ data.weather }}</p>
      <p>Feels like: {{ Math.round(data.feelsLike) }}°</p>
    </span>
      <p>Wind: 
        <span>
          <img :style="`transform:rotate(${data.windDeg}deg)`" class="wind-arrow" src="../assets/arrow.png" alt="arrow">
        </span> {{ data.windSpeed }} meters per second
      </p>
    <div>
      
    </div>
    <p>Humidity: {{ data.humidity }}%</p>
    <p>Pressure: {{ data.pressure }}hPa</p>
  </div>

  </div>
</template>

<script>

export default {
  name: 'MainData',
  data: () => ({
    city: '',
    countriesData: [],
    cities: [],
    filteredCities: [],
    modal: false,
    locationKey: '691a16990bb9a70332e47bcd8fedfcf6',
    weatherKey: 'b3159520889c26993eea38da6ac0ea9d',
    index: '',
    weatherData: [{
    }],
  }),
  methods: {
    deleteCity(index) {
      alert(`${this.weatherData[index].name} has been removed!`)
      this.weatherData.splice(index, 1)
    },

    filterCities() {
      this.filteredCities = this.cities.filter(city => {
        return city.toLowerCase().startsWith(this.city.toLowerCase())
      })
    },

    setCity(city) {
      this.city = city
      this.modal = false
    },

    onEnter(city) {
      this.city = city
      this.modal = false
    },

    addToFavourite(id, name) {
      if(localStorage.length > 4) {
        alert('Oops, that\'s too much! 5 cities maximum. Remove another city!')
        return
      } 
      alert(`${name} was added to favourite!`)
      localStorage.setItem(id, name)
    },

    removeFromFavourite(id, name) {
      alert(`${name} was removed from favourite!`)
      localStorage.removeItem(id, name)
    },
    
    async getWeather() {
      let lat
      let lon
      await(await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${this.city}&appid=${this.locationKey}`)
        .then((response) => response.json())
          .then((data) => {lat = data[0].lat, lon = data[0].lon}))

      let weather = await(await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&exclude=minutely,hourly,daily,alerts&appid=${this.weatherKey}`)
        .then((response) => response.json())
          .then((result) => result))
          
      this.weatherData[0].id = weather.id
      this.weatherData[0].name = weather.name
      this.weatherData[0].country = weather.sys.country
      this.weatherData[0].temp = weather.main.temp
      this.weatherData[0].weather = weather.weather[0].description
      this.weatherData[0].feelsLike = weather.main.feels_like
      this.weatherData[0].windSpeed = weather.wind.speed
      this.weatherData[0].windDeg = weather.wind.deg
      this.weatherData[0].humidity = weather.main.humidity
      this.weatherData[0].pressure = weather.main.pressure
      this.weatherData[0].favourite = false
      if(localStorage.getItem(this.weatherData[0].id, this.weatherData[0].name)) {
        this.weatherData[0].favourite = true
      }
    },

    async addCity() {
      let lat
      let lon
      ++this.index
      
      await(await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${this.city}&appid=${this.locationKey}`)
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
        favourite: false
      })
      if(localStorage.getItem(this.weatherData[this.index].id, this.weatherData[this.index].name)) {
        this.weatherData[this.index].favourite = true
      }
      this.city = ''
    },
  },

  async created() {

    let countriesData = []
    let citiesArr = []
    let cities = []
    await(await fetch('https://countriesnow.space/api/v0.1/countries')
      .then(response => response.json())
        .then(response => countriesData.push(response.data))
          .catch(err => console.error(err)))
    
    function getCities(arr) {
      for(let i = 0; i < arr.length; i++) {
          for(let j = 0; j < arr[i].length; j++) {
            citiesArr.push(arr[i][j].cities)
          }
      }
      for(let i = 0; i < citiesArr.length; i++) {
        cities.push(...citiesArr[i])
      }
      
      cities = Array.from(new Set(cities))
      return cities
    }
    
    this.cities = getCities(countriesData).sort()
  },

  async mounted() {
    
    await(await fetch('http://ip-api.com/json/?fields=city')
      .then((response) => response.json())
        .then((result) => this.city = Object.values(result)))
    
    let lat
    let lon
    await(await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${this.city}&appid=${this.locationKey}`)
      .then((response) => response.json())
        .then((data) => {lat = data[0].lat, lon = data[0].lon}))

    let weather = await(await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&exclude=minutely,hourly,daily,alerts&appid=${this.weatherKey}`)
      .then((response) => response.json())
        .then((result) => result))
    this.weatherData.splice(0, 1)
    this.index = 0
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
      favourite: false
    })
    
    if(localStorage.getItem(this.weatherData[0].id, this.weatherData[0].name)) {
      this.weatherData[0].favourite = true
    }
    this.city = ''
  }
}
</script>

<style scoped>

  input:disabled {
    border-color: rgba(16, 16, 16, 0.3);
  }
  .autocomplete {
    display: block;
    margin: 17px;
  }

  .autocomplete-item {
    width: 200px;
  }
  
  .btn-block {
    display: flex;
    justify-content: space-between;
  }

  .remove-btn {
    background-color: transparent;
    border: 4px solid lightblue;
    border-radius: 50%;
    font-family: cursive;
    font-weight: bold;
    cursor: pointer;
  }

  .favourite-remove-btn {
    border: 4px solid gold;
  }

  .regular-btn {
    background-color: transparent;
    border: 4px solid lightblue;
    border-radius: 10px;
    font-weight: bold;
    cursor: pointer;
  }
  .favourite-btn {
    border: 4px solid gold;
  }

  .card {
    display: inline-block;
    padding: 10px 20px;
    width: 250px;
    margin: 40px;
    border: 4px solid lightblue;
    border-radius: 10px;
    text-align: left;
    font-size: 14px;
  }

  .favourite-item {
    border: 4px solid gold;
    border-radius: 10px;
  }

  .first-block {
    display: flex;
    justify-content: space-between;
    font-size: 24px;
    font-weight: bold;
  }

  .second-block {
    display: flex;
    justify-content: space-between;    
    text-align: start;
    font-style: italic;
    font-size: 20px;
  }

  .city {
    text-align: start;
    cursor: pointer;
  }

  .city:hover {
    text-decoration: underline;
  }

  .weather {
    margin-right: 5px;
  }

  .weather::first-letter {
    text-transform: uppercase;
  }

  .wind-arrow {
    width: 20px;
    vertical-align: middle;
    padding: 0 10px;
  }

  .variant-select {
    margin-top: -18px;
    margin-bottom: 18px;
    width: 208px;
  }

  .variant {
    list-style: none;
    padding: 5px;
    cursor: pointer;
    font-size: 14px;
  }

  .variant:hover {
    text-decoration: underline;
  }

</style>