<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <input :value="prefCode" @change="updatePrefCode" type="text" placeholder="E.g. 000"> -
    <input :value="cityCode" @change="updateCityCode" type="text" placeholder="E.g. 0000">
    <button id="postalCodeSubmit" v-on:click="getTownName">SUBMIT</button>
    <h1>Your town is: {{currentTown}}</h1>
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  data: () => ({
    base_url: 'https://madefor.github.io/postal-code-api/api/v1',
    currentTown: "",
    postalCode: "",
    enteredPostalCode: "",
    prefCode: "",
    cityCode: "",
  }),
   methods: {
    updatePrefCode(event){
      this.prefCode = event.target.value.trim()
      this.postalCode = this.prefCode + "-" + this.cityCode
    },
    updateCityCode(event){
      this.cityCode = event.target.value.trim()
      this.postalCode = this.prefCode + "-" + this.cityCode
    },
    getTownName: async function() {
      console.log("the PS is:" + this.postalCode)
      console.log("the PC is:" + this.prefCode)
      console.log("the CC is:" + this.cityCode)
      const response = await fetch(`${this.base_url}/${this.prefCode}/${this.cityCode}.json`,{
        method: 'GET',
        headers: {
          'accept': 'application/json' ,
        },
      })
      const json = await response.json()
      console.log(json, "res")
      this.currentTown = json.data[0].en.prefecture + ", " + json.data[0].en.address1 + ", "+ json.data[0].en.address2
      
    },
},
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
