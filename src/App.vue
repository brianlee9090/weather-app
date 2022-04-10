<template>
  <div id="app">
  <c-text v-show="!checkPostCode">Welcome to the weather app. <br>
  Please enter your Japan postal code. </c-text>
    <div class="inputBlock">
      <c-input width="20%" class="input" :value="prefCode" @change="updatePrefCode" type="tel"  maxlength="3" variant="filled" placeholder="E.g: 000" size="medium" /> -
      <c-input width="20%" class="input" :value="cityCode" @change="updateCityCode" type="tel" maxlength="4" variant="filled" placeholder="E.g: 0000" size="medium"/>
      <c-button id="postalCodeSubmit" variant="outline" variant-color="blue" size="md" v-on:click="getGeoWeatherData()">Check</c-button>
    </div>
  <c-text v-show="wrongPostalCode" color="red">Please enter a valid Japan postal code. </c-text>
  <div v-show="!wrongPostalCode">
    <div class="townBlock">
    <c-text v-show="checkPostCode" fontSize="5vh" fontWeight="bolder" >{{currentTown}}</c-text>
    </div>
      <div class="infoBlock">
      <div class="textblock">
        <h5 class="blockIntro" v-show="checkPostCode">3 day forcast</h5>
      </div>
      <div id="weather">
        <Weather id="weatherContent" :weatherStuff="weatherData" />
      </div>
     </div>

     <div v-show="checkMapUrl" class="infoBlock">
      <div class="textblock">
        <h5 class="blockIntro">Area map</h5>
      </div>
        <img id="map" :key="mapUrl" :src="mapUrl" alt="googleMap">
    </div>

     <div v-show="checkNews" class="infoBlock">
      <div class="textblock">
        <h5 class="blockIntro">News today in {{this.cityNameEN}}</h5>
      </div>
      <News id="newsContent" :newsStuff="news" />
     </div>
    </div>
  </div>

</template>

<script>
import Weather from './components/Weather.vue'
import News from './components/News.vue'
import { CText, CInput, CButton} from '@chakra-ui/vue'

export default {
  name: 'App',
  components: {
    Weather, News, CText, CInput, CButton
  },
  data: () => ({
    base_url: 'https://madefor.github.io/postal-code-api/api/v1',
    currentTown: "",
    postalCode: "",
    enteredPostalCode: "",
    prefCode: "",
    cityCode: "",
    cityNameJP: "",
    cityNameEN: "",
    checkPostCode: false,
    checkMapUrl: false,
    checkNews: false,
    coordinates: "",
    map: "",
    mapUrl: "",
    lat: "",
    lon: "",
    weatherData: [
    ],
    news: {
    newsTitle:"",
    newsIconUrl:"",
    newsUrl:""},
    wrongPostalCode: false,
  }),
   methods: {
    updatePrefCode(event){
      this.prefCode = event.target.value.trim()
      this.postalCode = this.prefCode + this.cityCode
    },
    updateCityCode(event){
      this.cityCode = event.target.value.trim()
      this.postalCode = this.prefCode + "-" + this.cityCode
    },
    getTownName: async function() {
      const response = await fetch(`${this.base_url}/${this.prefCode}/${this.cityCode}.json`,{
        method: 'GET',
        headers: {
          'accept': 'application/json' ,
        },
      })
      
      if (response.ok){
      const json = await response.json()
      this.currentTown = json.data[0].en.prefecture + ", " + json.data[0].en.address1 + ", "+ json.data[0].en.address2
      this.cityNameJP = json.data[0].ja.prefecture + json.data[0].ja.address1
      this.cityNameEN = json.data[0].en.prefecture + ", " + json.data[0].en.address1
      this.checkPostCode = true
      this.wrongPostalCode = false
      } else if(!response.ok){
        this.handlePostalCodeError()
      }
    },
    handlePostalCodeError(){
      this.wrongPostalCode = true
    },
    updateMapUrl(){
      this.mapUrl = `https://maps.googleapis.com/maps/api/staticmap?center=${this.postalCode}&zoom=15&size=800x450&key=${process.env.VUE_APP_gApiKey}`
      this.checkMapUrl = true
    },
    updateLatLong: async function(){
      const response = await fetch(`http://api.openweathermap.org/geo/1.0/zip?zip=${this.postalCode},JP&appid=${process.env.VUE_APP_wApiKey}`, {
        method: 'GET',
        headers: {
          'accept': 'application/json',
        }
      })
      const json = await response.json()
      this.lat = json.lat
      this.lon = json.lon
    },
    convertToDate(unix){
      return new Date(unix * 1000).toLocaleDateString('en-US', {weekday: 'short', month: 'short', day: 'numeric'})
    },
    getWeather: async function(){
      const response = await fetch(`https://api.openweathermap.org/data/2.5/onecall?lat=${this.lat}&lon=${this.lon}&units=metric&exclude=hourly,current,minutely,alerts&appid=${process.env.VUE_APP_wApiKey}`, {
        method: 'GET',
        headers: {
          'accept': 'application/json',
        }
      })
      const json = await response.json()
      const newWeatherData = []
      for(let i=0; i <=2; i++){
        newWeatherData.push(
          { date: this.convertToDate(json.daily[i].dt),
            weather: json.daily[i].weather[0].main,
            max: Math.round(json.daily[i].temp.max),
            min: Math.round(json.daily[i].temp.min),
            icon: json.daily[i].weather[0].icon
          }
        )
      }
      this.weatherData = newWeatherData
    },
    getGeoWeatherData: async function(){
      await this.getTownName(); 
      await this.updateMapUrl(); 
      await this.updateLatLong();
      this.getWeather();
      this.getNews();
    },
    getNews: async function(){
      const response = await fetch(`https://newsapi.org/v2/everything?q=${this.cityNameJP}&apiKey=${process.env.VUE_APP_nApiKey}`)
      const json = await response.json()
      console.log(json, "articles")
      this.news.newsTitle = json.articles[0].title
      this.news.newsIconUrl = json.articles[0].urlToImage
      this.news.newsUrl = json.articles[0].url
      this.checkNews = true
    }
    },
  
}
</script>


<style scoped>
#app {
  font-size: 16px;
  font-size: 2vw;
  display: flex;
  flex-direction: column;
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  justify-items: center;
  align-items: center;
  max-width: 70vw;
  max-height: 70vh;
  margin: auto;
}
.inputBlock{
  display:flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
.input{
  border-radius: 5px;
}
#postalCodeSubmit{
  margin-left: 5px;
}
#map {
  max-height: 45vh;
  border-radius: 15px;
}
.mapblock{
  padding: 0;
  margin: 0;
  display: inline-block;
}

h5{
  font-weight: bolder;
}
#weather{
  display: flex;
  justify-items: center;
  margin: 0px;
  font-size: calc(14px+1.5vw);
}
.infoBlock{
  display: flex;
  flex-direction: column;
  justify-items: flex-start;
  align-items: center;
  margin-top: 10px;
  margin-bottom: 5px;
  width: 100%;
}
.textblock{
  display: inline-block;
  width: 100%
}
#town{
  font-size: calc(100px, 25vh)
}
.blockIntro{
  text-align: start;
  margin-bottom: 8px;
}
</style>
