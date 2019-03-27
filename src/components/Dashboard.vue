<template>
  <div>
    <div v-if="loading" class="progress">
      <div class="indeterminate"></div>
    </div>
    <div class="container">
      <Departure :departures="departures.arbejde" direction="arbejde"/>
    </div>
    <div class="container">
      <Departure :departures="departures.hjem" direction="hjem"/>
    </div>
    <div id="status"></div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import axios, { AxiosPromise, AxiosRequestConfig } from "axios";
import Departure from "./Departure.vue";

@Component({
  components: {
    Departure
  }
})
export default class Dashboard extends Vue {
  private loading: boolean = true;

  getHours() {
    return new Date().getHours();
  }
  private departures: any = {
    hjem: [],
    arbejde: []
  };

  // TODO: use a better life cycle
  updated() {
    this.loading = false;
  }

  private defaultConf: AxiosRequestConfig = {
    headers: {
      "X-Requested-With": "XMLHttpRequest"
    }
  };

  mounted() {
    this.getDeparture(
      "https://cors-anywhere.herokuapp.com/http://xmlopen.rejseplanen.dk/bin/rest.exe/departureBoard?id=008600764&useTog=1&useBus=0&useMetro=0&offsetTime=0&format=json",
      "arbejde",
      3,
      (d: any) => {
        return (
          d.name === "A" &&
          (d.direction === "Køge St." ||
            d.direction === "Hundige St." ||
            d.direction === "Solrød Strand St.")
        );
      }
    );
    this.getDeparture(
      "https://cors-anywhere.herokuapp.com/http://xmlopen.rejseplanen.dk/bin/rest.exe/departureBoard?id=008600650&useTog=1&useBus=0&useMetro=0&offsetTime=0&format=json",
      "hjem",
      3,
      (d: any) => {
        return d.name === "A" && d.direction === "Hillerød St.";
      }
    );
  }

  getDeparture = async (
    url: string,
    key: string,
    limit: number,
    filter: Function
  ) => {
    //url = `data/${key}.json`;
    const response = await this.getData(url);
    if (response) {
      const departures = response.data.DepartureBoard.Departure;
      const filtered = departures.filter(filter);
      this.departures[key].push(...filtered);
      if (this.departures[key].length > limit) {
        this.departures[key].length = limit;
      }
    }
  };

  getData = async (url: string) => {
    try {
      return await axios.get(url, this.defaultConf);
    } catch (error) {
      console.error("Error loading JSON ", error);
      // TODO: do this the vue way
      const elm = document.getElementById("status");
      if (elm) {
        elm.innerHTML = error;
      }
    }
  };
}
</script>

<style scoped>
#status {
  padding: 5px;
  color: #c0c0c0;
}

.progress {
  background-color: #c0c0c0;
}

.indeterminate {
  background-color: gray;
}
</style>