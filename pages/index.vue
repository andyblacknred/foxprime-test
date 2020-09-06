<template>
  <v-app id="inspire">
    <v-app id="inspire">
      <v-navigation-drawer
        v-model="drawer"
        app
      >
        <v-list dense>
          <v-list-item>
            <v-btn block @click="clearData">Clear all data</v-btn>
          </v-list-item>
          <v-list-item>
            <v-btn block @click="saveToFile('uploadFile', 'application/json')">Save data to file</v-btn>
          </v-list-item>
          <v-list-item>
            <v-btn block @click="$el.querySelector('#fileInput').click()">Load data from file</v-btn>
          </v-list-item>
          <v-list-item>
            <input id="fileInput" class="d-none" type="file" @change="readFromFile($event)">
          </v-list-item>
        </v-list>
      </v-navigation-drawer>
      <v-app-bar
        app
        color="indigo"
        dark
      >
        <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
        <v-toolbar-title>Application</v-toolbar-title>
      </v-app-bar>
      <v-main>
        <v-container
          fluid
        >
          <v-row>
            <v-col cols="12" sm="12" md="12">
              <v-row
                align="start"
                justify="center"
              >
                <v-col cols="12" sm="12" md="9">
                  <v-text-field
                    label="Search"
                    outlined
                    placeholder="Enter city name here"
                    :error-messages="searchErrorMessage"
                    v-model="city"
                    @input="requestToApi"
                  ></v-text-field>
                </v-col>
                <v-col
                  cols="12"
                  sm="12"
                  md="3">
                  <v-btn block x-large @click="addToList">add</v-btn>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="12" sm="12" md="12" class="text-center text-sm-left">
              <v-row
                align="center"
                justify="center"
                v-if="current"
              >
                <v-col cols="12" sm="12" md="12">
                  <h1>Current city: {{ current.name }}</h1>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="12" sm="12" md="12" class="text-center text-sm-left">
              <v-row
                align="start"
                justify="center"
                v-if="current"
              >
                <v-col cols="12" sm="6" md="3">
                  Temperature: {{ parseInt(current.main.temp - 273.15) }}
                </v-col>
                <v-col cols="12" sm="6" md="3">
                  Weather: {{ current.weather[0].description }}
                </v-col>
                <v-col cols="12" sm="6" md="3">
                  Wind (deg): {{ current.wind.deg }}
                </v-col>
                <v-col cols="12" sm="6" md="3">
                  Wind (speed): {{ current.wind.speed }}
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="12" sm="12" md="12">
              <v-row
                align="start"
                justify="center"
                v-if="cityList.length"
              >
                <v-simple-table
                  fixed-header
                  height="300"
                  class="table-responsive"
                >
                  <thead>
                    <tr>
                      <th>City</th>
                      <th>Temperature</th>
                      <th>Weather</th>
                      <th>Wind (deg)</th>
                      <th>Wind (speed)</th>
                      <th>Remove</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(item, index) in cityList" :key="index">
                      <td>{{ item.name }}</td>
                      <td>{{ parseInt(item.main.temp - 273.15) }}</td>
                      <td>{{ item.weather[0].description }}</td>
                      <td>{{ item.wind.deg }}</td>
                      <td>{{ item.wind.speed }}</td>
                      <td><v-btn block @click="removeFromList(index)">Remove</v-btn></td>
                    </tr>
                  </tbody>
                </v-simple-table>
              </v-row>
            </v-col>
          </v-row>
          <v-dialog
            v-model="errorDialog"
            width="500"
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                color="red lighten-2"
                dark
                v-bind="attrs"
                v-on="on"
                class="d-none"
                id="errorMessageBtn"
              >
                Open error message
              </v-btn>
            </template>
            <v-card>
              <v-card-title class="headline grey lighten-2">
                {{ errorMessage }}
              </v-card-title>
              <v-divider></v-divider>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="primary"
                  text
                  @click="errorDialog = false"
                >
                  Ok
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-container>
      </v-main>
      <v-footer
        color="indigo"
        app
      >
        <span class="white--text">&copy; {{ new Date().getFullYear() }}</span>
      </v-footer>
    </v-app>
  </v-app>
</template>

<script>
import AppLogo from '~/components/AppLogo.vue'

export default {
  components: {
    AppLogo
  },
  data: function () {
    return {
      drawer: null,
      errorDialog: false,
      errorMessage: '',
      apiKey: 'd10dc92eb40b53618efda6c3206ec22d',
      current: false,
      city: 'Kyiv',
      cityList: [],
      searchErrorMessage: ''
    }
  },
  methods: {
    requestToApi: function () {
      this.$axios.$get(`https://api.openweathermap.org/data/2.5/weather?q=${ this.city }&appid=${ this.apiKey }`)
        .then((res) => {
          this.current = res;
          this.searchErrorMessage = '';
        }).catch((err) => {
          this.current = false;
          this.searchErrorMessage = 'No matches found';
      });
    },
    addToList: function () {
      if (this.current) {
        this.cityList.push(this.current);
        localStorage.setItem('cityList', JSON.stringify({ cityList: this.cityList }))
      }
    },
    removeFromList: function (index) {
      this.cityList.splice(index, 1);
      localStorage.setItem('cityList', JSON.stringify({ cityList: this.cityList }));
    },
    readFromFile: function (e) {
      let files = e.target.files;
      if (files.length === 0) {
        this.errorMessage = 'No file is selected';
        this.$el.querySelector('#errorMessageBtn').click();
        this.errorDialog = true;
        return;
      }
      let reader = new FileReader();
      let that = this;
      reader.onload = function(event) {
        let jsonFromFile = event.target.result[0] === '{' ? JSON.parse(event.target.result) : '';
        if(jsonFromFile.foxprimeFileArray) {
          that.cityList = that.cityList.concat(jsonFromFile.foxprimeFileArray);
          localStorage.setItem('cityList', JSON.stringify({ cityList: that.cityList }));
        } else {
          that.errorMessage = 'It`s not our file';
          that.$el.querySelector('#errorMessageBtn').click();
          that.errorDialog = true;
        }
        e.target.value = '';
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
    if(localStorage.getItem('cityList') === '' || localStorage.getItem('cityList') === 'undefined' || !localStorage.getItem('cityList') || localStorage.getItem('cityList') === '[object Object]') {
      localStorage.removeItem("cityList");
      localStorage.setItem('cityList', JSON.stringify({ cityList: this.cityList }));
    } else {
      this.cityList = JSON.parse(localStorage.getItem('cityList')).cityList;
    }
    this.requestToApi();
  },
  destroyed() {
    localStorage.setItem('cityList', this.cityList);
  }
}
</script>
