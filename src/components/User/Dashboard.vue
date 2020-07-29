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
                :items="['All', 'Beruwala', 'Kalutara', 'Agalawatta', 'Horana', 'Bandaragama', 'Panadura', 'Mathugama', 'Bulathsinhala']"
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
  var satelliteLayre
  var divisionLayerGroup
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
      handleSatelliteView () {
        if (document.getElementById('satelliteView').checked === true) {
          mymap.addLayer(satelliteLayre)
        } else {
          mymap.removeLayer(satelliteLayre)
        }
      },
      handleDivisionView () {
        console.log('ldsjf' + document.getElementById('divisionView').checked)
        if (document.getElementById('divisionView').checked === true) {
          mymap.addLayer(divisionLayerGroup)
        } else {
          mymap.removeLayer(divisionLayerGroup)
        }
      },
      changeDSCode: async function changeDSCode (choice) {
        var db = firebase.firestore()
        const cityRef = db.collection('votes')
        var dataRef = await cityRef.get()
        switch (choice) {
          case 'All': this.resetAndFillMaps(dataRef.docs, [6.5854, 80.1077], 11, 0, 0); break
          case 'Kalutara': this.resetAndFillMaps(this.filterWith(dataRef, ['21']), [6.589834, 79.96317], 12, 1321, 2); break
          case 'Beruwala': this.resetAndFillMaps(this.filterWith(dataRef, ['24']), [6.49310, 80.0222], 12, 1324, 1); break
          case 'Panadura': this.resetAndFillMaps(this.filterWith(dataRef, ['03']), [6.733761, 79.93200], 12, 1303, 6); break
          case 'Mathugama': this.resetAndFillMaps(this.filterWith(dataRef, ['30', '27']), [6.487791, 80.11434], 12, 1330, 7); break
          case 'Bandaragama': this.resetAndFillMaps(this.filterWith(dataRef, ['06', '18']), [6.72348, 79.9851], 13, 1306, 5); break
          case 'Horana': this.resetAndFillMaps(this.filterWith(dataRef, ['09', '10']), [6.753325, 80.06226], 13, 1309, 4); break
          case 'Agalawatta': this.resetAndFillMaps(this.filterWith(dataRef, ['03', '39', '36']), [6.4508063, 80.213525], 11, 1333, 3); break
          case 'Bulathsinhala': this.resetAndFillMaps(this.filterWith(dataRef, ['12', '15']), [6.656049, 80.17837], 12, 1312, 8); break
        }
      },
      filterWith: function filterWith (dataRef, dsCode) {
        var filteredVotes = dataRef.docs.filter(function (arrayItem) {
          if (dsCode === 0) {
            return true
          }
          if (dsCode.length === 1) {
            return arrayItem.data().ds.dsCode === dsCode[0]
          } else if (dsCode.length === 2) {
            return arrayItem.data().ds.dsCode === dsCode[0] || arrayItem.data().ds.dsCode === dsCode[1]
          } else if (dsCode.length === 3) {
            return arrayItem.data().ds.dsCode === dsCode[0] || arrayItem.data().ds.dsCode === dsCode[1] || arrayItem.data().ds.dsCode === dsCode[2]
          }
        })
        return filteredVotes
      },
      filterGNList: function filterGNList (gnList, electorateCode) {
        return gnList.filter(function (gnItems) {
          return gnItems.electorateCode === electorateCode
        })
      },
      drawMaps: async function drawMaps () {
        var db = firebase.firestore()
        const cityRef = db.collection('votes')
        var dataRef = await cityRef.get()
        this.fillMaps(dataRef.docs, [6.5854, 80.1077], 11, 0)
      },
      resetAndFillMaps: function resetAndFillMaps (data, mapCenter, zoomLevel, dsCode, electorateCode) {
        mymap.remove()
        this.fillMaps(data, mapCenter, zoomLevel, dsCode, electorateCode)
      },
      fillMaps: async function fillMaps (data, mapCenter, zoomLevel, dsCode, electorateCode) {
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
        satelliteLayre = L.tileLayer('http://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', {
          maxZoom: 20,
          subdomains: ['mt0', 'mt1', 'mt2', 'mt3']
        }).addTo(mymap)
        var command = L.control({position: 'topright'})
        command.onAdd = function (map) {
          var div = L.DomUtil.create('div', 'satelliteView')
          div.innerHTML = '<form style="background-color: white;width: 108px">' +
            '<input id="satelliteView" type="checkbox" style="margin: 5px" checked>Satellite View' +
            '<input id="divisionView" type="checkbox" style="margin: 5px" checked>Division View</form>'
          return div
        }
        command.addTo(mymap)
        mymap.createPane('locationMarker')
        mymap.getPane('locationMarker').style.zIndex = 500
        document.getElementById('satelliteView').addEventListener('click', this.handleSatelliteView, false)
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
          var point = L.circle([doc.data().latitude, doc.data().longitude], 1, {color: colorFill, weight: 8, fillColor: colorFill, opacity: 0.7, pane: 'locationMarker'})
          point.addTo(mymap).bindPopup(popupString).bringToFront()
        })
        var filteredGNList = gnList
        if (dsCode !== 0) {
          filteredGNList = this.filterGNList(gnList, electorateCode)
        }
        divisionLayerGroup = L.layerGroup()
        for (var polygonCoordinate in filteredGNList) {
          data = filteredGNList[polygonCoordinate].boundary_detail.coordinates
          var polygonData = {
            type: 'Polygon',
            coordinates: data
          }
          var divisionLayer = L.geoJSON(polygonData)
          divisionLayer.setStyle({ weight: '1', opacity: 0.5 }).bindPopup(filteredGNList[polygonCoordinate].gnd_name)
          divisionLayerGroup.addLayer(divisionLayer)
          document.getElementById('divisionView').addEventListener('click', this.handleDivisionView, false)
        }
        divisionLayerGroup.addTo(mymap)

        // All vote pie chart
        CanvasJS.addColorSet('greenShades',
          [
            slpfaColor,
            unpColor,
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
          axisX: {
            interval: 1,
            labelMaxWidth: 100,
            labelFontSize: 10
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
