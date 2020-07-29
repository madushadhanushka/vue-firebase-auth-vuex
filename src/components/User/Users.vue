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
      <td class="text-xs-left">{{ props.item.voteCount }}</td>
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
        { text: 'GN', value: 'gn' },
        { text: 'Vote Count', value: 'voteCount' }
      ],
      userList: []
    }
  },
  mounted: async function mounted () {
    var db = firebase.firestore()
    const userRef = db.collection('users')
    const votesRef = db.collection('votes')
    var voteDataRef = await votesRef.get()
    var userDataRef = await userRef.get()
    var user = []
    for (var userVoteIndex in userDataRef.docs) {
      user.push(userDataRef.docs[userVoteIndex].data())
      user[userVoteIndex].voteCount = 0
      for (var voteIndex in voteDataRef.docs) {
        if (userDataRef.docs[userVoteIndex].data().nic === voteDataRef.docs[voteIndex].data().nic) {
          user[userVoteIndex].voteCount++
        }
      }
    }
    console.log(user)
    for (var userIndex in user) {
      this.userList.push({
        name: user[userIndex].name,
        nic: user[userIndex].nic,
        ds: user[userIndex].ds.dsName,
        gn: user[userIndex].gn.gnName,
        voteCount: user[userIndex].voteCount
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
