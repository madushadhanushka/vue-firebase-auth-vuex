<template>
  <v-container>
  <v-layout>
    <v-flex xs12 sm12>
      <v-card>
        <v-card-text>
          <div class="text-md-center" v-if="user">
            <v-icon large color="green darken-2">check_circle</v-icon>
              <h1 class="green--text">Login Success</h1>
              <h4 class="headline mb-0"><b class="red--text">Email :</b> {{ user.email }}</h4>
          </div>
        </v-card-text>
     </v-card>
    </v-flex>
  </v-layout>
    <div id="chartContainer" style="height: 370px; width: 100%;"></div>
    <div id="mapid" style="width: 80%; height: 80vh;"></div>
  </v-container>
</template>

<script>
  var L = window.L
  var CanvasJS = window.CanvasJS
  export default {
    computed: {
      user () {
        return this.$store.getters.user
      }
    },
    mounted () {
      var chart = new CanvasJS.Chart('chartContainer', {
        animationEnabled: true,
        theme: 'light2', // "light1", "dark1", "dark2"
        title: {
          text: 'Vote Percentage in Kaluthara'
        },
        axisY: {
          interval: 10,
          suffix: '%'
        },
        toolTip: {
          shared: true
        },
        data: [{
          type: 'stackedBar100',
          toolTipContent: '{label}<br><b>{name}:</b> {y} (#percent%)',
          showInLegend: true,
          name: 'April',
          dataPoints: [
            { y: 600, label: 'Water Filter' },
            { y: 400, label: 'Modern Chair' },
            { y: 120, label: 'VOIP Phone' },
            { y: 250, label: 'Microwave' },
            { y: 120, label: 'Water Filter' },
            { y: 374, label: 'Expresso Machine' },
            { y: 350, label: 'Lobby Chair' }
          ]
        },
        {
          type: 'stackedBar100',
          toolTipContent: '<b>{name}:</b> {y} (#percent%)',
          showInLegend: true,
          name: 'May',
          dataPoints: [
              { y: 400, label: 'Water Filter' },
              { y: 500, label: 'Modern Chair' },
              { y: 220, label: 'VOIP Phone' },
              { y: 350, label: 'Microwave' },
              { y: 220, label: 'Water Filter' },
              { y: 474, label: 'Expresso Machine' },
              { y: 450, label: 'Lobby Chair' }
          ]
        },
        {
          type: 'stackedBar100',
          toolTipContent: '<b>{name}:</b> {y} (#percent%)',
          showInLegend: true,
          name: 'June',
          dataPoints: [
              { y: 300, label: 'Water Filter' },
              { y: 610, label: 'Modern Chair' },
              { y: 215, label: 'VOIP Phone' },
              { y: 221, label: 'Microwave' },
              { y: 75, label: 'Water Filter' },
              { y: 310, label: 'Expresso Machine' },
              { y: 340, label: 'Lobby Chair' }
          ]
        }]
      })
      chart.render()

      var mymap = L.map('mapid', {
        center: [6.5854, 80.0907],
        zoom: 12
      })
      L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw', {
        maxZoom: 100,
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
          '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
          'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        id: 'mapbox.streets'
      }).addTo(mymap)
      L.circle([6.5854, 80.0907], 1, {color: 'red', weight: 8, fillColor: 'blue'}).addTo(mymap)
    }
  }
</script>
