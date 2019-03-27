<template>
  <div class="hello">
    <h1>{{new Date().toLocaleTimeString() }}</h1>
    <h2>På arbejde...</h2>
    <ul
      v-for="(departure,index) in departures.hjem"
      :key="`${index}-${departure.JourneyDetailRef.ref}`"
    >
      <li>
        <span>{{ departure.time }}</span>
        <span style="color: red">{{departure.rtTime ? ` ${departure.rtTime}` : ""}}</span>
      </li>
    </ul>
    <p/>
    <h2>Hjem fra arbejde...</h2>
    <ul
      v-for="(departure,index) in departures.arbejde"
      :key="`${index}-${departure.JourneyDetailRef.ref}`"
    >
      <li>
        <span>{{ departure.time }}</span>
        <span style="color: red">{{departure.rtTime ? ` ${departure.rtTime}` : ""}}</span>
      </li>
    </ul>
    <p/>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import axios, { AxiosPromise, AxiosRequestConfig } from "axios";

@Component
export default class HelloWorld extends Vue {
  @Prop() private msg!: string;

  private departures: any = {
    hjem: [],
    arbejde: []
  };

  //xhr.setRequestHeader("Origin", 'maximum.blog');

  private defaultConf: AxiosRequestConfig = {
    headers: {
      "X-Requested-With": "XMLHttpRequest"
    }
  };

  mounted() {
    this.getDeparture(
      "https://cors-anywhere.herokuapp.com/http://xmlopen.rejseplanen.dk/bin/rest.exe/departureBoard?id=008600764&useTog=1&useBus=0&useMetro=0&offsetTime=0&format=json",
      "arbejde",
      (d: any) => {
        return d.name === "A" && d.direction === "Køge St.";
      }
    );
    this.getDeparture(
      "https://cors-anywhere.herokuapp.com/http://xmlopen.rejseplanen.dk/bin/rest.exe/departureBoard?id=008600650&useTog=1&useBus=0&useMetro=0&offsetTime=0&format=json",
      "hjem",
      (d: any) => {
        return d.name === "A" && d.direction === "Hillerød St.";
      }
    );
  }

  getDeparture = async (url: string, key: string, filter: Function) => {
    const response = await this.getData(url);
    if (response) {
      const departures = response.data.DepartureBoard.Departure;
      const filtered = departures.filter(filter);
      this.departures[key].push(...filtered);
    }
  };

  getData = async (url: string) => {
    try {
      return await axios.get(url, this.defaultConf);
    } catch (error) {
      console.error("Error loading JSON ", error);
    }
  };
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
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
</style>
