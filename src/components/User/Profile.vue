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
              <v-select
                :items="['All', 'Kalutara', 'Beruwala', 'Panadura', 'Mathugama','Bandaragama', 'Horana', 'Ingiriya', 'Walallavita', 'Agalawatta', 'Bulathsinhala', 'Dodangoda', 'Madurawala', 'Palindanuwara', 'Millaniya']"
                label="Select District Secretariat Division"
                v-on:change="changeDSCode"
                solo
              ></v-select>
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
  import gnList from '../../assets/gn_list'

  var L = window.L
  var CanvasJS = window.CanvasJS
  var slpfaColor = '#ff4F4F'
  var unpColor = '#08ff06'
  var otherColor = '#cbcbff'
  var mymap
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
      changeDSCode: async function changeDSCode (choice) {
        var db = firebase.firestore()
        const cityRef = db.collection('votes')
        var dataRef = await cityRef.get()
        switch (choice) {
          case 'All': this.resetAndFillMaps(dataRef.docs, [6.5854, 80.1077], 11, 0); break
          case 'Kalutara': this.resetAndFillMaps(this.filterWith(dataRef, '21'), [6.589834, 79.96317], 12, 1321); break
          case 'Beruwala': this.resetAndFillMaps(this.filterWith(dataRef, '24'), [6.49310, 80.0222], 12, 1324); break
          case 'Panadura': this.resetAndFillMaps(this.filterWith(dataRef, '03'), [6.733761, 79.93200], 12, 1303); break
          case 'Mathugama': this.resetAndFillMaps(this.filterWith(dataRef, '30'), [6.487791, 80.11434], 12, 1330); break
          case 'Bandaragama': this.resetAndFillMaps(this.filterWith(dataRef, '06'), [6.72348, 79.9851], 13, 1306); break
          case 'Horana': this.resetAndFillMaps(this.filterWith(dataRef, '09'), [6.753325, 80.06226], 13, 1309); break
          case 'Ingiriya': this.resetAndFillMaps(this.filterWith(dataRef, '10'), [6.763266, 80.17232], 13, 1310); break
          case 'Walallavita': this.resetAndFillMaps(this.filterWith(dataRef, '39'), [6.423396, 80.20112], 12, 1339); break
          case 'Bulathsinhala': this.resetAndFillMaps(this.filterWith(dataRef, '12'), [6.656049, 80.17837], 12, 1312); break
          case 'Dodangoda': this.resetAndFillMaps(this.filterWith(dataRef, '27'), [6.56205, 80.05187], 12, 1327); break
          case 'Madurawala': this.resetAndFillMaps(this.filterWith(dataRef, '15'), [6.659044, 80.09365], 12, 1315); break
          case 'Palindanuwara': this.resetAndFillMaps(this.filterWith(dataRef, '36'), [6.505204, 80.27508], 12, 1336); break
          case 'Millaniya': this.resetAndFillMaps(this.filterWith(dataRef, '18'), [6.65033, 80.0329], 12, 1318); break
        }
      },
      filterWith: function filterWith (dataRef, dsCode) {
        var filteredVotes = dataRef.docs.filter(function (arrayItem) {
          return arrayItem.data().ds.dsCode === dsCode
        })
        return filteredVotes
      },
      filterGNList: function filterGNList (gnList, dsCode) {
        return gnList.filter(function (gnItems) {
          return gnItems.ds_division_code === dsCode
        })
      },
      drawMaps: async function drawMaps () {
        var db = firebase.firestore()
        const cityRef = db.collection('votes')
        var dataRef = await cityRef.get()
        this.fillMaps(dataRef.docs, [6.5854, 80.1077], 11, 0)
      },
      resetAndFillMaps: function resetAndFillMaps (data, mapCenter, zoomLevel, dsCode) {
        mymap.remove()
        this.fillMaps(data, mapCenter, zoomLevel, dsCode)
      },
      fillMaps: async function fillMaps (data, mapCenter, zoomLevel, dsCode) {
        var slpfaCount = 0
        var unpCount = 0
        var otherCount = 0
        let voteCandidateData = []
        for (var dataItem in data) {
          for (var vote in data[dataItem].data().votes) {
            var voteCandIndex = -1
            for (var i in voteCandidateData) {
              if (voteCandidateData[i].label === data[dataItem].data().party.toUpperCase() + ' ' + data[dataItem].data().votes[vote]) {
                voteCandIndex = i
                break
              }
            }
            if (voteCandIndex === -1) {
              var color = otherColor
              if (data[dataItem].data().party === 'slpfa') {
                color = slpfaColor
              } else if (data[dataItem].data().party === 'unp') {
                color = unpColor
              }
              voteCandidateData.push({'label': data[dataItem].data().party.toUpperCase() + ' ' + data[dataItem].data().votes[vote], 'y': 1, 'color': color})
            } else {
              voteCandidateData[voteCandIndex]['y']++
            }
          }
          if (data[dataItem].data().party === 'slpfa') {
            slpfaCount += data[dataItem].data().votes.length
          } else if (data[dataItem].data().party === 'unp') {
            unpCount += data[dataItem].data().votes.length
          } else {
            otherCount += data[dataItem].data().votes.length
          }
        }
        // Draw map
        mymap = L.map('mapid', {
          center: mapCenter,
          zoom: zoomLevel
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
        var filteredGNList = gnList
        console.log('filtersli1', filteredGNList)
        if (dsCode !== 0) {
          filteredGNList = this.filterGNList(gnList, dsCode)
        }
        console.log('filtersli', filteredGNList)
        for (var polygonCoordinate in filteredGNList) {
          data = filteredGNList[polygonCoordinate].boundary_detail.coordinates
          var polygonData = {
            type: 'Polygon',
            coordinates: data
          }
          var layer2 = L.geoJSON(polygonData)
          layer2.setStyle({ weight: '1', opacity: 0.5 }).bindPopup(filteredGNList[polygonCoordinate].gnd_name)
          mymap.addLayer(layer2)
        }

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
      this.drawMaps()
    }
  }
</script>
