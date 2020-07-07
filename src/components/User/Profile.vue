<template>
  <v-container>
  <v-layout>
    <v-flex xs12 sm12>
      <v-card>
        <v-card-text>
          <div class="text-md-center" v-if="user">
            <v-avatar>
              <img
                src="static/img/avatar.png"
                alt="John"
              >
            </v-avatar>
              <h5 class="headline mb-0"><b class="red--text">Email :</b> {{ user.email }}</h5>
          </div>
        </v-card-text>
     </v-card>
    </v-flex>
  </v-layout>

    <div id="chartContainer" style="height: 370px; width: 100%;"></div>
    <div id="mapid" style="width: 100%; height: 80vh;"></div>
  </v-container>
</template>

<script>
  import * as firebase from 'firebase'

  var L = window.L
  var CanvasJS = window.CanvasJS
  export default {
    computed: {
      user () {
        return this.$store.getters.user
      }
    },
    methods: {
      fillMaps: async function fillMaps () {
        var db = firebase.firestore()
        const cityRef = db.collection('votes')
        var data = await cityRef.get()
        var slpfa = []
        var unp = []
        var other = []
        var voteCount
        data.forEach(doc => {
          if (doc.data().party === 'slpfa') {
            voteCount = this.getCountForParty(slpfa, doc.data().votes[0])
            if (voteCount === 0) {
              slpfa.push({y: 1, label: doc.data().votes[0]})
            }
          } else if (doc.data().party === 'unp') {
            voteCount = this.getCountForParty(unp, doc.data().votes[0])
            if (voteCount === 0) {
              unp.push({y: 1, label: doc.data().votes[0]})
            }
          } else {
            voteCount = this.getCountForParty(other, doc.data().votes[0])
            if (voteCount === 0) {
              other.push({y: 1, label: doc.data().votes[0]})
            }
          }
        })
        console.log(slpfa, unp, other)
        // Draw chart
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
            name: 'SLPFA',
            color: '#ff4444',
            dataPoints: slpfa
          }, {
            type: 'stackedBar100',
            toolTipContent: '<b>{name}:</b> {y} (#percent%)',
            showInLegend: true,
            name: 'UNP',
            color: 'green',
            dataPoints: unp
          }, {
            type: 'stackedBar100',
            toolTipContent: '<b>{name}:</b> {y} (#percent%)',
            showInLegend: true,
            name: 'Others',
            color: 'lightblue',
            dataPoints: other
          }]
        })
        chart.render()
        // Draw map
        var mymap = L.map('mapid', {
          center: [8.1662979, 80.968543],
          zoom: 17
        })
        L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw', {
          maxZoom: 100,
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
            '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
            'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          id: 'mapbox.streets'
        }).addTo(mymap)
        data.forEach(doc => {
          var colorFill
          if (doc.data().party === 'slpfa') {
            colorFill = 'red'
          } else if (doc.data().party === 'unp') {
            colorFill = 'green'
          }
          L.circle([doc.data().latitude, doc.data().longitude], 1, {color: colorFill, weight: 8, fillColor: colorFill, opacity: 0.5})
            .addTo(mymap).bindPopup('<b>' + doc.data().name + '</b><br />' + doc.data().party)
        })
      },
      getCountForParty: function getCountForParty (array, match) {
        for (var i = 0; i < array.length; i++) {
          if (array[i].label === match) {
            array[i].y++
            return array
          }
        }
        return 0
      }
    },
    mounted: function mounted () {
      this.fillMaps()
    }
  }
</script>
