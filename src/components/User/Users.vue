<template>
  <v-container>
    <v-data-table
      :headers="headers"
      :items="userList"
      class="elevation-1"
      hide-actions
    ><template v-slot:items="props">
      <td>{{ props.item.name }}</td>
      <td class="text-xs-left">{{ props.item.nic }}</td>
      <td class="text-xs-left">{{ props.item.ds }}</td>
      <td class="text-xs-left">{{ props.item.gn }}</td>
    </template></v-data-table>
  </v-container>
</template>
<script>
import * as firebase from 'firebase'
export default {
  data () {
    return {
      headers: [
        {
          text: 'Name',
          align: 'start',
          sortable: false,
          value: 'name'
        },
        { text: 'NIC', value: 'nic' },
        { text: 'DS', value: 'ds' },
        { text: 'GN', value: 'gn' }
      ],
      userList: []
    }
  },
  mounted: async function mounted () {
    var db = firebase.firestore()
    const userRef = db.collection('users')
    var dataRef = await userRef.get()
    for (var userIndex in dataRef.docs) {
      this.userList.push({
        name: dataRef.docs[userIndex].data().name,
        nic: dataRef.docs[userIndex].data().nic,
        ds: dataRef.docs[userIndex].data().ds.dsName,
        gn: dataRef.docs[userIndex].data().gn.gnName
      })
    }
  },
  methods: {
    search () {
      return ''
    }
  }
}
</script>
