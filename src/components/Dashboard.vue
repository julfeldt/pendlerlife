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

    <div>Home: {{atHome}} | Work: {{atWork}}</div>
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
  private PROXY_URL: string = "https://cors-anywhere.herokuapp.com";
  private BASE_URL: string = `${
    this.PROXY_URL
  }/http://xmlopen.rejseplanen.dk/bin/rest.exe`;
  private loading: boolean = true;
  private atHome: boolean = false;
  private atWork: boolean = false;

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

  getCoords(onCoords: Function) {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        position => {
          // Get current cordinates.
          const positionCords = {
            latitude: position.coords.latitude,
            longitude: position.coords.longitude
          };
          onCoords(positionCords);
        },
        error => {
          // On error code..
          console.log("Error ", error);
        },
        { timeout: 30000, enableHighAccuracy: true, maximumAge: 75000 }
      );
    }
  }

  addToStatus(status: string) {
    const elm = document.getElementById("status");
    if (elm) {
      elm.innerHTML += "<br/>" + status;
    }
  }

  mounted() {
    this.getCoords((coords: any) => {
      this.isNearStation(coords.latitude, coords.longitude, "Friheden St.")
        .then(res => {
          this.atHome = res;
        })
        .catch(error => {
          this.addToStatus(" - H_ERROR: " + error);
        });

      this.isNearStation(coords.latitude, coords.longitude, "Østerport St.")
        .then(res => {
          this.atWork = res;
        })
        .catch(error => {
          this.addToStatus(" - A_ERROR: " + error);
        });
    });

    // Get departure to home
    this.getDeparture(
      `${
        this.BASE_URL
      }/departureBoard?id=008600764&useTog=1&useBus=0&useMetro=0&offsetTime=0&format=json`,
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

    // Get departure to work
    this.getDeparture(
      `${
        this.BASE_URL
      }/departureBoard?id=008600650&useTog=1&useBus=0&useMetro=0&offsetTime=0&format=json`,
      "hjem",
      3,
      (d: any) => {
        return d.name === "A" && d.direction === "Hillerød St.";
      }
    );
  }

  // Get the actual departure
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

  // Returns true if the given coords are in the vicinity of the specified station
  isNearStation = async (x: number, y: number, q: string) => {
    const response = await this.getData(
      `${
        this.BASE_URL
      }/stopsNearby?coordX=${x}&coordY=${y}&maxRadius=3000&maxNumber=30&format=json`
    );
    if (response) {
      const stops = response.data.LocationList.StopLocation;
      return stops.some((s: any) => s.name === q);
    } else {
      const elm = document.getElementById("status");
      if (elm) {
        elm.innerHTML = "kan ikke finde GPS";
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