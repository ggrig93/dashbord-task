<template>
  <div id="app">
    <div>
      <h2>Single File</h2>
      <hr/>
      <label>File
        <input type="file" ref="file" @change="handleFileUpload( $event )"/>
      </label>
      <select v-model="selected" @change="changeCountry(selected)">
        <option v-for="c in country" :key="c">{{c}}</option>
      </select>
    </div>
    <apexchart v-if="table" :key="key" width="1200" type="line" :options="chartOptions" :series="series"></apexchart>
  </div>
</template>

<script>
// import axios from 'axios';
// import * as d3 from 'd3'
export default {
  name: 'App',
  data() {
    return {
      file: '',
      key: 0,
      selected: '',
      country: null,
      chartOptions: {
        xaxis: {
          categories: []
        }
      },
      series: [
        {
          name: '',
          data: []
        }
      ],
      table: null
    }
  },
  methods: {
    handleFileUpload() {
      this.file = this.$refs.file.files[0];
      const reader = new FileReader()
      const vm = this
      reader.onload = function(e) {
        const text = e.target.result;
        function csvToArray(str, delimiter = ",") {
          const headers = str.slice(0, str.indexOf("\n")).split(delimiter);
          const rows = str.slice(str.indexOf("\n") + 1).split("\n");
          const arr = rows.map(function (row) {
            const values = row.split(delimiter);
            const el = headers.reduce(function (object, header, index) {
              object[header] = values[index];
              return object;
            }, {});
            return el;
          });
          vm.table = arr
          vm.country = new Set(arr.map(el => el.country))
          return arr;
        }
        csvToArray(text);
      };
      reader.readAsText(this.file)
    },
    changeCountry(country) {
      this.chartOptions.xaxis.categories = new Set(this.table.filter(el => el.country === country).map(e => new Date(e.chandged_at).toLocaleDateString()))
      this.series = [{name: country, data: this.table.filter(el => el.country === country).map(e => e.rate)}]
      this.key = 1 - this.key
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
