<template>
  <div id="app">
    <div class="inputBlock">
      <c-input class="input" :value="prefCode" @change="updatePrefCode" type="tel"  maxlength="3" variant="filled" placeholder="  000" size="medium" minH="5vh" height="80%" minWidth="3vw" maxWidth="6vw"/> -
      <c-input class="input" :value="cityCode" @change="updateCityCode" type="tel" maxlength="4" variant="filled" placeholder="  0000" size="medium" minH="5vh" height="80%" minWidth="4vw" maxWidth="7vw"/>
      <c-button id="postalCodeSubmit" variant="outline" variant-color="blue" size="md" minH="5vh" height=80% maxH="7vh" maxW="10vw" v-on:click="getGeoWeatherData()"  >🔍</c-button>
    </div>
    <div class="forgotBlock">
      <c-text class="reminder" color="grey">Look up a postal code</c-text> 
      <a :href="`https://www.post.japanpost.jp/zipcode/`">
      <c-button variant="ghost" variant-color="blue" size="md" minH="5vh" height=80% maxH="7vh" maxW="10vw" ><c-icon name="question-outline" size="18px"></c-icon></c-button></a>
    </div>
  <div class="messageBlock" v-show="wrongPostalCode">
  <img class="messageIcon" :src="`https://tsukatte.com/wp-content/uploads/2019/01/hatena.png`">
  <c-text color="red">Please enter a valid Japan postal code. </c-text>
  </div>
    <div class="messageBlock" v-show="!checkPostCode">
    <img class="messageIcon" :src="`https://www.bhs-mizu.jp/common/uploads/mark-post-001-300x300.png`">
    <c-text id="welcomeText">Welcome to the weather app. <br>
  Please enter your Japan postal code. </c-text>
    </div>

    <div id="retrivedInfoChunk" v-show="!wrongPostalCode">
    <div class="townBlock">
    <c-text v-show="checkPostCode" textAlign="start" fontSize="1.8em" fontWeight="bolder" >{{currentTown}}</c-text>
    </div>
      <div class="infoBlock">
      <div class="textblock">
        <h5 class="blockIntro" v-show="checkPostCode">3-day forecast</h5>
      </div>
      <div id="weather">
        <Weather id="weatherContent" :weatherStuff="weatherData" />
      </div>
     </div>

     <div v-show="checkMapUrl" class="infoBlock">
      <div class="textblock">
        <h5 class="blockIntro">Area map</h5>
      </div>
        <div id="map">
         <a :href="`http://maps.google.com/maps?z=12&t=m&q=loc:${this.lat}+${this.lon}`">
         <img id="mapContent" :key="mapUrl" :src="mapUrl" alt="googleMap">
         </a>
        </div>
    </div>
     <div v-show="checkMapUrl" class="infoBlock">
      <div class="textblock">
        <h5 class="blockIntro">News topic in the neighborhood</h5>
      </div>
      <div id="news">
        <p v-show="showAPIError">Please use run the app from localhost or change your News API plan.</p>
        <News v-show="checkNews" id="newsContent" :newsStuff="news" />
      </div>
     </div>
    </div>
  </div>

</template>

<script>
import Weather from './components/Weather.vue'
import News from './components/News.vue'
import { CText, CInput, CButton, CIcon} from '@chakra-ui/vue'

export default {
  name: 'App',
  components: {
    Weather, News, CText, CInput, CButton, CIcon
  },
  data: () => ({
    base_url: 'https://madefor.github.io/postal-code-api/api/v1',
    currentTown: "",
    postalCode: "",
    enteredPostalCode: "",
    prefCode: "",
    cityCode: "",
    prefNameJP: "",
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
    showAPIError: false,
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
      this.checkPostCode = true
      if (response.ok){
      const json = await response.json()
      this.currentTown = json.data[0].en.prefecture + ", " + json.data[0].en.address1 + ", "+ json.data[0].en.address2
      this.prefNameJP = json.data[0].ja.prefecture
      this.cityNameJP = json.data[0].ja.prefecture + json.data[0].ja.address1
      this.cityNameEN = json.data[0].en.prefecture + ", " + json.data[0].en.address1
      
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
      const response = await fetch(`https://api.openweathermap.org/geo/1.0/zip?zip=${this.postalCode},JP&appid=${process.env.VUE_APP_wApiKey}`, {
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
      let response = await fetch(`https://newsapi.org/v2/everything?q=${this.cityNameJP}&domains=asahi.com&apiKey=${process.env.VUE_APP_nApiKey}`)
      let json = await response.json()
      if (json.articles.length ===0){
        console.log("no town article")
        response = await fetch(`https://newsapi.org/v2/everything?q=${this.prefNameJP}&domains=asahi.com&apiKey=${process.env.VUE_APP_nApiKey}`)
        json = await response.json()
      }
      console.log(json, "articles")
      const number = Math.floor(Math.random()*json.articles.length)
      this.news.newsTitle = json.articles[number].title
      this.news.newsIconUrl = json.articles[number].urlToImage
      this.news.newsUrl = json.articles[number].url
      if (json.status !== "ok"){
        this.showAPIError = true;
      }
      if (json.status === "ok"){
      this.checkNews = true}
    }
    },
  
}
</script>


<style scoped>
#app {
  font-size: 14px;
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
  max-width: 50vw;
  max-height: 70vh;
  margin: auto;
  margin-top:30px;
}
.inputBlock{
  width: 100%;
  display:flex;
  flex-direction: row;
  justify-content: flex-end;
  align-items: center;
  margin-bottom: 5px;
}
.input{
  border-radius: 5px;
  margin-left: 5px;
  margin-right: 5px;
}
.forgotBlock{
  width: 100%;
  display:flex;
  flex-direction: row;
  justify-content: flex-end;
  align-items: center;
  margin-bottom: 10px;
}
.messageBlock{
  margin: auto;
  margin-top: 28vh;
  display: flex;
  flex-direction: column;
  justify-items: center;
}
.messageIcon{
  margin: auto;
  margin-bottom: 12px;
  max-height: 18vh;
  max-width: 18vw
}

#postalCodeSubmit{
  margin-left: 5px;
}

#map{
  padding: 0;
  margin: 0;
  max-height: 100%;
  max-width:50vw;
}
#mapContent {
  width: 100%;
  height: 100%;
  border-radius: 7px;
}

h5{
  font-weight: bolder;
}

.infoBlock{
  display: flex;
  flex-direction: column;
  justify-items: flex-start;
  align-items: center;
  margin-top: 5vh;
  margin-bottom: 5px;
  width: 100%;
  
}
.textblock{
  display: inline-block;
  width: 100%;
  
}
.townBlock{
  margin-top: 5vh;
}

.blockIntro{
  font-size: 1.2em;
  text-align: start;
  margin-bottom: 8px;
}

#news{
  max-width: 50vw;
  margin-bottom: 80px;
}

#weather{
  width: 100%;
  font-size: 0.8em;
  justify-items: space-between
}
#weatherContent{
  
}
</style>
