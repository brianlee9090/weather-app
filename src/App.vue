<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <input :value="enteredPostalCode" @change="updatePostalCode" type="text" placeholder="Please enter your 7 digit postal code">
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
    base_url: 'https://postal-codes-jp.azurewebsites.net/api/PostalCodes/',
    currentTown: "",
    postalCode: "",
    enteredPostalCode: 0,
  }),
   methods: {
    updatePostalCode(event){
      this.enteredPostalCode = event.target.value.trim()
      this.postalCode = this.enteredPostalCode
    },
    getTownName: async function() {
      const response = await fetch(`${this.base_url}/${this.postalCode}`,{
        method: 'GET',
        headers: {
          'accept': 'application/json' ,
        },
      })
      const json = await response.json()
      this.currentTown = json[0].city.pref.name + json[0].city.name
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
