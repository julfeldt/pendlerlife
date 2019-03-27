<template>
  <div class="container departure">
    <table>
      <thead>
        <tr>
          <th>Afgange > {{direction}}</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(departure,index) in departures"
          :key="`${index}-${departure.JourneyDetailRef.ref}`"
        >
          <td>
            <span>{{departure.time }}</span>
            <span class="delay">{{getDelayString(departure)}}</span>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import axios, { AxiosPromise, AxiosRequestConfig } from "axios";
import moment from "moment";

@Component
export default class Departure extends Vue {
  @Prop() direction!: string;
  @Prop() departures!: any[];

  getDelayString(departure: any): string {
    const delay = this.getDelayInMinutes(departure);
    return delay > 0 ? `(+${delay})` : "";
  }

  getDelayInMinutes(departure: any): number {
    let delayMinutes: number = 0;
    if (departure.rtTime) {
      delayMinutes =
        moment(departure.rtTime, "HH:mm").diff(
          moment(departure.time, "HH:mm")
        ) / 60000;
    }
    return delayMinutes;
  }
}
</script>

<style scoped>
.delay {
  color: #c0c0c0;
  padding-left: 5px;
}

.departure {
  padding: 10px;
}
</style>
