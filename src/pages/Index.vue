<template>
  <q-page class="flex flex-center">
    <!-- <img
      alt="Quasar logo"
      src="~assets/quasar-logo-full.svg"
    > -->
    <q-card dark bordered class="bg-primary my-card" style="width: 40%;height: 88%;margin-top:10px;margin-left:3%;margin-bottom:20px;">
      <q-card-section>
        <div class="text-h6 text-white">登陆房间</div>
        <!-- <div class="text-subtitle2">by John Doe</div> -->
      <!--   <q-form
	      @submit="onSubmit"
	      class="q-gutter-md"
	    > -->
	      <q-input
	        filled
	        v-model="room_id"
	        label="Room"
	        hint="Room id"
	        lazy-rules
	        label-color="grey-10"
          color="grey-10"
          bg-color="grey-1"
	        :rules="[ val => val && val.length > 0 || 'Please type something']"
	      />

	      <q-toggle v-model="accept" label="I accept the license and terms" color="white" keep-color />

	      <div>
	        <q-btn label="登陆" @click="onSubmit" color="primary"/>
          <q-btn label="创建" type="createNew" color="primary" @click="onCreateNew"/>
	      </div>
	    <!-- </q-form> -->
      </q-card-section>
  </q-card>
  </q-page>
</template>

<script>
export default {
 name: 'PageIndex',
 data () {
 	return {
 		accept: false,
 		room_id: ''
 	}
 },


methods: {

onSubmit () {
  if (this.accept !== true) {
    this.$q.notify({
      color: 'red-5',
      textColor: 'white',
      icon: 'warning',
      message: 'You need to accept the license and terms first'
    });
  } else {
    if (this.room_id) {
      this.$store.commit('hashnum/updateRoomHash', this.room_id);
      this.$router.push({path:"/room"});
    } else {
      this.$q.notify({
        color: 'red-5',
        textColor: 'white',
        icon: 'warning',
        message: 'You need to input the room id to log into an existed room!'
      });
    }
    this.$q.notify({
      color: 'green-4',
      textColor: 'white',
      icon: 'cloud_done',
      message: 'Submitted'
    })
  }
},

onCreateNew () {
  if (this.accept !== true) {
    this.$q.notify({
      color: 'red-5',
      textColor: 'white',
      icon: 'warning',
      message: 'You need to accept the license and terms first'
    });
  } else {
    
    let val = Math.floor(Math.random() * 0xFFFFFF).toString(16);
    let val_str = val.substring(1);
    this.room_id = val_str;
    this.$store.commit('hashnum/updateRoomHash', this.room_id);
    
    this.$q.notify({
      color: 'green-4',
      textColor: 'white',
      icon: 'cloud_done',
      message: 'Create new room successfully!'
    })
    this.$router.push({path:"/room"})
  }
}


}
}

</script>
