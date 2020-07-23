<template>
  <div class="container">
    <div class>
      <h1 class="text-center">World Time</h1>
      <div class="alert alert-warning" role="alert" v-if="error">{{error}}</div>
      <label class>Select an area</label>
      <model-list-select
        :list="areas"
        v-model="area"
        option-value="key"
        option-text="value"
        placeholder="Select an area"
        @input="onChange($event)"
      ></model-list-select>
    </div>
    <div>
      <label class="text-gray-400 pl-0">Select a location</label>
      <model-list-select
        :list="locations"
        v-model="location"
        option-value="key"
        option-text="value"
        placeholder="Select an area"
        @input="locationChange($event)"
      ></model-list-select>
    </div>
    <div class="text-center mt-5">
      <h5>The time in this location</h5>
      <h1>{{time| formatDate}}</h1>
    </div>
    <div class="text-center">
      <button class="btn btn-primary" @click="reset($event)">Reset form</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ModelListSelect } from "vue-search-select";
import "vue-search-select/dist/VueSearchSelect.css";
let uri_api = "http://worldtimeapi.org/api/timezone";
export default {
  name: "HelloWorld",
  components: {
    ModelListSelect,
  },
  data() {
    return {
      areas: [],
      locations: [],
      area: {},
      location: {},
      selectedarea: null,
      selectedlocation: null,
      time: null,
      error: "",
    };
  },
  mounted: function () {
    var self = this;
    axios
      .get(uri_api)
      .then(
        function (response) {
          response.data.map(function (value) {
            self.areas.push({ key: value, value: value });
          });
        }.bind(this)
      )
      .catch(function (error) {
        this.error = error;
      });
  },
  methods: {
    onChange(event) {
      var self = this;
      this.selectedarea = this.areas.filter(function (el) {
        return el.key == event.key;
      });
      this.selectedarea = this.selectedarea[0];
      console.log("this.selected", this.selectedarea.value);
      let value = this.selectedarea.value.slice(
        0,
        this.selectedarea.value.indexOf("/")
      );
      console.log("this.selected", value);

      axios
        .get(uri_api + "/" + value)
        .then(
          function (response) {
            response.data.map(function (value) {
              self.locations.push({ key: value, value: value });
            });
          }.bind(this)
        )
        .catch(function (error) {
          this.error = error;
        });
    },
    locationChange(event) {
      this.selectedlocation = this.locations.filter(function (el) {
        return el.key == event.key;
      });
      this.selectedlocation = this.selectedlocation[0];

      axios
        .get(uri_api + "/" + this.selectedlocation.value)
        .then(
          function (response) {
            this.time = response.data.datetime;
            console.log("response", this.time);
          }.bind(this)
        )
        .catch(function (error) {
          this.error = error;
        });
    },
    reset() {
      this.area = {};
      this.location = {};
      this.selectedarea = null;
      this.selectedlocation = null;
      this.time = null;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
label {
  text-align: left;
}
</style>
