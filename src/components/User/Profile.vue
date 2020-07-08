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
            <v-col class="d-flex" cols="12" sm="6">
              <v-select
                :items="['All', 'Kalutara', 'Beruwala', 'Panadura', 'Mathugama','Bandaragama', 'Horana', 'Ingiriya', 'Walallavita', 'Agalawatta', 'Bulathsinhala', 'Dodangoda', 'Madurawala', 'Palindanuwara', 'Millaniya']"
                label="Select District Secretariat Division"
                solo
              ></v-select>
            </v-col>
          </div>
        </v-card-text>
     </v-card>
    </v-flex>
  </v-layout>
    <br/>
    <div id="allVotePieChart" style="height: 600px; width: 100%;"/>
    <br/>
    <div id="allVoteCandidateBarChart" style="height: 600px; width: 100%;"/>
    <br/>
    <div id="mapid" style="width: 100%; height: 80vh;"></div>
  </v-container>
</template>

<script>
  import * as firebase from 'firebase'

  var L = window.L
  var CanvasJS = window.CanvasJS
  var slpfaColor = '#ff4F4F'
  var unpColor = '#08ff06'
  var otherColor = '#cbcbff'
  export default {
    computed: {
      user () {
        return this.$store.getters.user
      }
    },
    methods: {
      sortDps: function sortDps (dps) {
        do {
          var swap = false
          for (var i = 0; i < dps.length - 1; i++) {
            if (dps[i].y > dps[i + 1].y) {
              var temp = dps[i].y
              dps[i].y = dps[i + 1].y
              dps[i + 1].y = temp
              swap = true
            }
          }
        }
        while (swap)
      },
      fillMaps: async function fillMaps () {
        var db = firebase.firestore()
        const cityRef = db.collection('votes')
        var data = await cityRef.get()
        var slpfaCount = 0
        var unpCount = 0
        var jvpCount = 0
        var otherCount = 0
        let voteCandidateData = []
        for (var dataItem in data.docs) {
          for (var vote in data.docs[dataItem].data().votes) {
            var result = voteCandidateData.filter(obj => {
              return obj.label === data.docs[dataItem].data().party + ' ' + data.docs[dataItem].data().votes[vote]
            })
            var voteCandIndex = -1
            for (var i in voteCandidateData) {
              if (voteCandidateData[i].label === data.docs[dataItem].data().party.toUpperCase() + ' ' + data.docs[dataItem].data().votes[vote]) {
                voteCandIndex = i
                break
              }
            }
            if (voteCandIndex === -1) {
              var color = otherColor
              if (data.docs[dataItem].data().party === 'slpfa') {
                color = slpfaColor
              } else if (data.docs[dataItem].data().party === 'unp') {
                color = unpColor
              }
              voteCandidateData.push({'label': data.docs[dataItem].data().party.toUpperCase() + ' ' + data.docs[dataItem].data().votes[vote], 'y': 1, 'color': color})
            } else {
              voteCandidateData[voteCandIndex]['y']++
            }
            console.log('result', result)
          }
          if (data.docs[dataItem].data().party === 'slpfa') {
            slpfaCount += data.docs[dataItem].data().votes.length
          } else if (data.docs[dataItem].data().party === 'unp') {
            unpCount += data.docs[dataItem].data().votes.length
          } else if (data.docs[dataItem].data().party === 'jvp') {
            jvpCount += data.docs[dataItem].data().votes.length
          } else {
            otherCount += data.docs[dataItem].data().votes.length
          }
        }
        console.log('voteCandidateData', voteCandidateData)
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
          console.log(doc.data())
          var popupString = '<b>' + doc.data().name + '</b><br />' + doc.data().party.toUpperCase()
          if (doc.data().votes.length > 0) {
            popupString += '<br/>Vote 1: ' + doc.data().votes[0]
          }
          if (doc.data().votes.length > 1) {
            popupString += '<br/>Vote 2: ' + doc.data().votes[1]
          }
          if (doc.data().votes.length > 2) {
            popupString += '<br/>Vote 3: ' + doc.data().votes[2]
          }
          L.circle([doc.data().latitude, doc.data().longitude], 1, {color: colorFill, weight: 8, fillColor: colorFill, opacity: 0.7})
            .addTo(mymap).bindPopup(popupString)
        })
        // All vote pie chart
        CanvasJS.addColorSet('greenShades',
          [
            slpfaColor,
            unpColor,
            '#ff0303',
            otherColor
          ])
        var allVotePieChart = new CanvasJS.Chart('allVotePieChart',
          {
            colorSet: 'greenShades',
            title: {
              text: 'Overall result'
            },
            legend: {
              maxWidth: 350,
              itemWidth: 120
            },
            data: [
              {
                type: 'pie',
                showInLegend: true,
                legendText: '{indexLabel}',
                dataPoints: [
                  { y: slpfaCount, indexLabel: 'SLPFA' },
                  { y: unpCount, indexLabel: 'UNP' },
                  { y: jvpCount, indexLabel: 'JVP' },
                  // eslint-disable-next-line standard/object-curly-even-spacing
                  { y: otherCount, indexLabel: 'Others'}
                ]
              }
            ]
          })
        allVotePieChart.render()
        // Add Candidate bar chart
        var chart = new CanvasJS.Chart('allVoteCandidateBarChart', {
          title: {
            text: 'Candidate votes'
          },
          data: [
            {
              type: 'bar',
              dataPoints: voteCandidateData
            }
          ]
        })

// Sort the dataPoints in acending order by y values so that the longest bar moves to the top
        // this.sortDps(chart.options.data[0].dataPoints)

        chart.render()
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
