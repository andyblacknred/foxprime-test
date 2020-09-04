<template>
  <section class="container">
    <div>
      <app-logo/>
      <h1 class="title">
        foxprime-test
      </h1>
      <h2 class="subtitle">
        Nuxt.js project
      </h2>
      <div class="links">
        <a
          href="https://nuxtjs.org/"
          target="_blank"
          class="button--green">Documentation</a>
        <a
          href="https://github.com/nuxt/nuxt.js"
          target="_blank"
          class="button--grey">GitHub</a>
      </div>
    </div>
    <input type="file" @change="readFromFile($event)">
    <input type="text" v-model="city" @input="requestToApi">
    <button @click="addToList">add</button>
    <button @click="saveToFile('uploadFile', 'application/json')">Save to file</button>
    <button @click="clearData">Clear data</button>
    <table>
      <tr>
        <th>City</th>
        <th>Temperature</th>
        <th>Weather</th>
        <th>Wind (deg)</th>
        <th>Wind (speed)</th>
        <th>Remove</th>
      </tr>
      <tr v-for="(item, index) in cityList" :key="index">
        <td>{{ item.name }}</td>
        <td>{{ parseInt(item.main.temp - 273.15) }}</td>
        <td>{{ item.weather[0].description }}</td>
        <td>{{ item.wind.deg }}</td>
        <td>{{ item.wind.speed }}</td>
        <td><button @click="removeFromList(index)">Remove</button></td>
      </tr>
    </table>
  </section>
</template>

<script>
import AppLogo from '~/components/AppLogo.vue'

export default {
  components: {
    AppLogo
  },
  data: function () {
    return {
      apiKey: 'd10dc92eb40b53618efda6c3206ec22d',
      city: 'Kyiv',
      current: {},
      cityList: []
    }
  },
  methods: {
    requestToApi: function () {
      this.$axios.$get(`https://api.openweathermap.org/data/2.5/weather?q=${ this.city }&appid=${ this.apiKey }`)
        .then((res) => {
          this.current = res;
          console.log(this.current);
        }).catch((err) => {
          console.log(err);
          this.current = false;
        console.log(this.current);
      });
    },
    addToList: function () {
      if (this.current) {
        this.cityList.push(this.current);
        localStorage.setItem('cityList', JSON.stringify({ cityList: this.cityList }))
      }
      console.log(this.cityList);
      console.log(JSON.stringify(this.cityList));
    },
    removeFromList: function (index) {
      this.cityList.splice(index, 1);
      localStorage.setItem('cityList', JSON.stringify({ cityList: this.cityList }));
    },
    readFromFile: function (e) {
      let files = e.target.files;
      if (files.length === 0) {
        console.log('No file is selected');
        return;
      }
      let reader = new FileReader();      let that = this;
      reader.onload = function(event) {
        let jsonFromFile = JSON.parse(event.target.result);
        if(jsonFromFile.foxprimeFileArray) {
          that.cityList = that.cityList.concat(jsonFromFile.foxprimeFileArray);
          localStorage.setItem('cityList', JSON.stringify({ cityList: that.cityList }));
        } else {
          console.log('It`s not our file');
        }
      };
      reader.readAsText(files[0]);
    },
    saveToFile: function (filename, type) {
      let data = JSON.stringify({
        name: "foxprimeFile",
        foxprimeFileArray: this.cityList
      });
      let file = new Blob([data], {type: type});
      if (window.navigator.msSaveOrOpenBlob) // IE10+
        window.navigator.msSaveOrOpenBlob(file, filename);
      else { // Others
        let a = document.createElement("a"),
          url = URL.createObjectURL(file);
        a.href = url;
        a.download = filename;
        document.body.appendChild(a);
        a.click();
        setTimeout(function() {
          document.body.removeChild(a);
          window.URL.revokeObjectURL(url);
        }, 0);
      }
    },
    clearData: function () {
      this.cityList = [];
      localStorage.removeItem('cityList');
    }
  },
  mounted() {
    if(localStorage.getItem('cityList').length === 0 || localStorage.getItem('cityList') === 'undefined') {
      localStorage.removeItem("cityList");
      localStorage.setItem('cityList', JSON.stringify({ cityList: this.cityList }));
    } else {
      this.cityList = JSON.parse(localStorage.getItem('cityList')).cityList;
    }
    this.requestToApi();
    console.log(localStorage.getItem('cityList').length);
  },
  destroyed() {
    localStorage.setItem('cityList', this.cityList);
  }
}
</script>

<style>
.container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; /* 1 */
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>

