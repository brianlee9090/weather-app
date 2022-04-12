<template>
  <div class="weather">
    <div class="dayBlock" v-for="(day,index) in weatherStuff" :key="`day`+index">
        <div class="dateBlock" :key="`date`+index">{{day.date}}</div>
        <div class="weatherInfoBlock" :key="`weatherInfo`+index">
        <div class="weatherBlock" :key="`weatherBlock`+index">
            <img class="weatherImg" :key="`img`+index" :src="`https://openweathermap.org/img/wn/${day.icon}@2x.png`">
            <label :key="`weather`+index">{{day.weather}} </label>
        </div>
        <div :key="`tempBlock`+index">
            <div :key="`high`+index">High: {{day.max}}</div>
            <div :key="`low`+index">Low: {{day.min}}</div>
        </div>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Weather',
  props: {
    weatherStuff: Array,
  },

//   http://openweathermap.org/img/wn/01d@2x.png
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.weather{
    margin:auto;
    display:flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-evenly;
    align-content: space-evenly;
}
.dayBlock{
    padding: 10px;
    border-radius: 15px;
    margin: 1vw;
    display: flex;
    flex-direction: column;
    justify-items: center;
     background-color: aliceblue;
}

.weatherBlock{
    display: flex;
    flex-direction: column;
    justify-items: center;
}

.weatherImg{
    margin: auto;
    width: 8vh;
    height: 8vh;
}
.dateBlock{
    font-weight: bold;
}
</style>
