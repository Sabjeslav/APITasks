<template>
  <div class="hello">
    <h1>{{ label }}</h1>

    <div id="main" v-on:change="calculate">
      <span> Convert from: </span>
      <select v-model="converted.convertFrom">
        <option v-bind:value="UAH"> {{ UAH.ccy }}</option>
        <option v-for="current in currents" v-bind:key="current.ccy" v-bind:value="current">{{ current.ccy }}</option>
      </select>
      <br>

      <span> Convert To: </span>
      <select v-model="converted.convertTo">
        <option v-bind:value="UAH"> {{ UAH.ccy }}</option>
        <option v-for="current in currents" v-bind:key="current.ccy" v-bind:value="current">{{ current.ccy }}</option>
      </select>
      <br>
      <span> Enter Amount: </span>
      <input type="number" min="1" v-model="converted.amount">
      <br>
      <span>{{ converted.amount }} {{ converted.convertFrom.ccy }} equals {{ converted.result }} {{ converted.convertTo.ccy }} </span>
    </div>
    <br>

    <h1>{{ label2 }}</h1>
    <div class="post">
      <div class="areaSelector">
        <span>Оберіть Область</span>
        <select v-model="selectedArea" v-on:change="getCities">
          <option v-for="area in areas" v-bind:key="area.Ref" v-bind:value="area.Ref">
            {{ area.Description }}
          </option>
        </select>
        <br>
        <span>Оберіть місто</span>
        <select v-model="searchCity.request" v-on:change="getWarehouses">
          <option value="" v-for="city in cities" v-bind:key="city.Ref">
            {{ city.Description }}
          </option>
        </select>
      </div>
      <br>
      <span>Оберіть Відділення</span>
      <div class="areaSelector">
        <select>
          <option v-for="warehouse in warehouses" v-bind:key="warehouse.Ref">
            {{ warehouse.Description }}
          </option>
        </select>
      </div>
    </div>
  </div>
</template>

<script>

import axios from 'axios'
const HOST = "api.novaposhta.ua/v2.0/json";
const API_KEY = "9a557481f95094531372a9d1b55222c8";

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data: function () {
    return {
      label: "Currency converter",
      label2: "Nova Poshta",
      currents: [],
      UAH: {
        ccy: "UAH",
        base_ccy: "UAH",
        buy: 1,
        sale: 1
      },
      converted: {
        convertFrom: {},
        convertTo: {},
        amount: 1,
        result: null
      },
      areas: [],
      selectedArea: null,
      cities: [],
      searchCity: {
        isSearching: false,
        request: ""
      },
      selectedCity: null,
      warehouses: [],
      selectedWarehouse: null
    }
  },

  created: function () {
    this.getAreas()
    //console.log(this.areas)
  },

  mounted: function () {
    axios.get("https://api.privatbank.ua/p24api/pubinfo?json&exchange&coursid=5").then((response) => {
      //console.log(response.data);
      this.currents = response.data;
    })
  },
  methods: {
    calculate: function () {
      const counted = ((parseFloat(this.converted.amount) * parseFloat(this.converted.convertFrom.sale)) / this.converted.convertTo.sale);
      if (this.converted.convertFrom.ccy === "RUR" && this.converted.convertTo.ccy === "BTC") this.converted.result = counted.toFixed(5);
      else this.converted.result = counted.toFixed(4);
    },
    getAreas: function () {
      axios.post(`https://${HOST}/`, {
        apiKey: API_KEY,
        modelName: "Address",
        calledMethod: "getAreas",
        methodProperties: {}
      })
          .then(response => {
            if (response.data.success)
              this.areas = response.data.data
            else
              console.error(response.data.errors);
          });
    },
    getCities: function () {
      this.selectedCity = null;
      this.searchCity.request = "";
      console.log(this.selectedArea);
      axios.post(`https://${HOST}/`, {
        "apiKey": API_KEY,
        "modelName": "Address",
        "calledMethod": "getCities",
        "methodProperties": {
          "AreaRef": this.selectedArea
        }
      })
          .then(response => {
            //console.log(response.data.data)
            if (response.data.success)
              this.cities = response.data.data
            else
              console.error(response.data.errors);
          });
    },
    getWarehouses: function () {
      this.selectedWarehouse = null;
      axios.post(`https://${HOST}/`, {
        "apiKey": API_KEY,
        "modelName": "AddressGeneral",
        "calledMethod": "getWarehouses",
        "methodProperties": {
          "CityRef": this.selectedCity.Ref
        }
      })
          .then(response => {
            console.log(response.data.data)
            if (response.data.success)
              this.warehouses = response.data.data
            else
              console.error(response.data.errors);
          });
    }
  }
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

#main {
  display: flex;
  margin-left: auto;
  margin-right: auto;
  justify-content: center;
  align-content: center;
  flex-direction: column;
  width: 30vw;
  height: 20vh;
}

select, input {
  width: 50%;
  border-radius: 10px;
  outline: none;
  display: flex;
  margin-left: auto;
  margin-right: auto;
}

.post {
  display: flex;
  margin-left: auto;
  margin-right: auto;

  flex-direction: column;
  align-content: center;
  justify-content: center;
  border: 1px solid black;
  width: 30vw;
  height: 20vh;
}
</style>
