<template>
  <section class="page">
    <q-btn class="absolute" style="top:80px; left:20px" color="primary" icon="arrow_back"
      @click="$router.push({ name: 'HomePage' })" />
    <h4 class="text-center">Logs device: {{ this.deviceId }}</h4>
    <div id="chart"></div>

    <table class="table">
      <thead>
        <tr>
          <th>Hora</th>
          <th>Temperatura</th>
          <th>Presión</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="log in this.logs" :key="log.id">
          <td>{{ this.parseDate(log.serverTimestamp) }}</td>
          <td>{{ this.parseTemperature(log.temperature) }}</td>
          <td>{{ this.parsePressure(log.pressure) }}</td>
        </tr>
      </tbody>
    </table>
  </section>
</template>

<script>
import { defineComponent, ref } from 'vue';
import ApexCharts from 'apexcharts'
export default defineComponent({
  name: 'LogsPage',
  components: {},
  data () {
    return {
      deviceId: '',
      logs: [],
      dates:[],
      temperatures:[],
      pressures:[],
      chart: {}
    };
  },
  async mounted () {
    this.deviceId = this.$route.params.deviceId;
    await this.getLogs();
    this.createChart()
  },
  async unmounted () {
    this.logs = [];
    //destroy chart
    this.chart.destroy();
  },
  methods: {
    async getLogs () {
      const response = await fetch(
        `http://localhost:3000/api/device/logs/${this.deviceId}`,
        {
          method: 'get',
          headers: {
            'content-type': 'application/json',
          },
        }
      );
      if (!response.ok) {
        // create error instance with HTTP status text
        const error = new Error(response.statusText);
        error.json = response.json();
        throw error;
      }
      const rawLogs = await response.json();
      this.logs = Object.values(rawLogs);
      // console.log(this.logs);
    },
    //parse date and time from utc date
    parseDate (date) {
      const d = new Date(date);
      const formattedDate = `${d.getDate()}/${d.getMonth() + 1}/${d.getFullYear()} ${d.getHours() < 10 ? '0' + d.getHours() : d.getHours()}:${d.getMinutes() < 10 ? '0' + d.getMinutes() : d.getMinutes()}:${d.getSeconds() < 10 ? '0' + d.getSeconds() : d.getSeconds()}`;
      this.dates.push(formattedDate)
      
      return formattedDate
    },
    //convert string to real number with 2 decimal
    parseTemperature (temperature) {
      temperature = parseFloat(temperature).toFixed(2);
      this.temperatures.push(temperature)
      return temperature
    },
    parsePressure (pressure) {
      pressure = parseFloat(pressure).toFixed(2);
      this.pressures.push(pressure)
      return pressure
    },
    createChart () {
      var options = {
        chart: {
          type: 'line',
        },
        series: [{
          name: 'Temperatura',
          data: this.temperatures
        
        },
        {
          name: 'Presión',
          data: this.pressures
        }
      ],
        xaxis: {
          categories: this.dates
        }
      }

      this.chart = new ApexCharts(document.querySelector("#chart"), options);

      this.chart.render();
    }

  },
});
</script>

<style lang="scss" scoped>
h3 {
  text-align: center;
}

.page {
  width: 90%;
  margin: 0 auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
  border: 1px solid #ddd;
}

.table thead {
  background-color: #f5f5f5;
}

.table thead th {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.table td {
  border: 1px solid #ddd;
  padding: 8px;
}
</style>
