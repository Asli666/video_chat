<template>
  <q-page padding>
    <div class="my-card fixed-left" style="width: 50%;height: 79%;margin-top:100px;margin-left:10%;margin-bottom:10%;border-style:groove;">
      <video id="remoteVideo" autoplay muted style="width: 98%; height: 98%;"></video>
    </div>
    <div class="my-card fixed-right" style="width: 25%;height: 28%;margin-top:100px;margin-right:12%;border-style:groove;">
      <video id="localVideo" autoplay muted style="width: 98%; height: 98%;"></video>
    </div>
    <q-card class="my-card fixed-bottom-right" style="width: 25%;height: 49%; bottom: 9%; right: 12%">
    	<q-card-section>
	        <div class="text-subtitle2">Room ID: {{this.roomName}}</div>
      	</q-card-section>
      	<q-separator/>
      	<q-scroll-area style="height: 60%;" ref="chatScroll">
        <div class="q-pa-md row justify-center">
		    <div v-for="(_data, i) in chat_data" style="width: 100%; max-width: 400px">
		      <q-chat-message
		        :name=_data[0]
		        :avatar=_data[1]
		        :text=[_data[2]]
		        :sent=_data[3]
		      />
		    </div>
  		</div>
  		</q-scroll-area>
  		<q-separator/>
  		<div class="q-pa-md q-gutter-sm" style="width: 104%; height: 23%;">
	    <q-editor
	      v-model="editor"
	      min-height="18%"

	      :definitions="{
	        send: {
	          tip: 'Send your message!',
	          icon: 'cloud_upload',
	          label: 'Send',
	          handler: sendWord
	        }
	      }"
	      :toolbar="[
	        ['bold', 'italic', 'strike', 'underline'],
	        ['send']
	      ]"
	    />
	    </div>
    </q-card>
  </q-page>
</template>
<script>
export default {
  name: 'PageName',
  data () {
 	return {
 		drone: null,
 		roomName: '',
 		room: null,
 		pc: null,
 		configuration: null,
 		editor: '',
 		dataChannel: null,
 		chat_data: [],
 	}
  },

  mounted: function () {
    
  this.$nextTick(function () {
  this.drone = new ScaleDrone('yiS12Ts5RdNhebyM');
  this.roomName = 'observable-' + this.$store.state.hashnum.roomHash;
  this.configuration = {
	  iceServers: [{
	    urls: 'stun:stun.l.google.com:19302'
	  }]
  };

  console.log('drone info:');
  console.log(this.drone)

  this.drone.on('open', error => {
	  if (error) {
	  	console.log('drone open error!')
	    return console.error(error);
	  }

	  console.log('drone open success!')
	  this.room = this.drone.subscribe(this.roomName);
	  this.room.on('open', error => {
	    if (error) {
	      console.log('room open error!')
	      this.onError(error);
	    }
	  });
	  // We're connected to the room and received an array of 'members'
	  // connected to the room (including us). Signaling server is ready.
	  this.room.on('members', members => {
	    console.log('MEMBERS', members);
	    console.log('RoomHash', this.roomName);
	    // If we are the second user to connect to the room we will be creating the offer
	    const isOfferer = members.length === 2;
	    this.startWebRTC(isOfferer);
	  });
  });
  })
  },

  methods: {

  	  sendMessage (message) {
  	  	  this.drone.publish({
    		  room: this.roomName,
    		  message
  		  });
  	  },

  	  startWebRTC(isOfferer) {
		  this.pc = new RTCPeerConnection(this.configuration);
		  console.log('PeerConn', this.pc)
		  // 'onicecandidate' notifies us whenever an ICE agent needs to deliver a
		  // message to the other peer through the signaling server
		  this.pc.onicecandidate = event => {
		    if (event.candidate) {
		      this.sendMessage({'candidate': event.candidate});
		    }
		  };

		  // If user is offerer let the 'negotiationneeded' event create the offer
		  if (isOfferer) {
		    this.pc.onnegotiationneeded = () => {
		      this.pc.createOffer().then(this.localDescCreated).catch(this.onError);
		    }
		    this.dataChannel = this.pc.createDataChannel('chat');
    		this.setupDataChannel();
		  } else {
		    this.pc.ondatachannel = event => {
		      this.dataChannel = event.channel;
		      this.setupDataChannel();
		    }
		  }

		  // When a remote stream arrives display it in the #remoteVideo element
		  this.pc.ontrack = event => {
		    const stream = event.streams[0];
		    if (!remoteVideo.srcObject || remoteVideo.srcObject.id !== stream.id) {
		      remoteVideo.srcObject = stream;
		    }
		  };

		  navigator.mediaDevices.getUserMedia({
		    audio: true,
		    video: true,
		  }).then(stream => {
		    // Display your local video in #localVideo element
		    localVideo.srcObject = stream;
		    // Add your stream to be sent to the conneting peer
		    stream.getTracks().forEach(track => this.pc.addTrack(track, stream));
		  }, this.onError);

		  // Listen to signaling data from Scaledrone
		  this.room.on('data', (message, client) => {
		    // Message was sent by us
		    if (client.id === this.drone.clientId) {
		      return;
		    }

		    if (message.sdp) {
		      // This is called after receiving an offer or answer from another peer
		      this.pc.setRemoteDescription(new RTCSessionDescription(message.sdp), () => {
		        // When receiving an offer lets answer it
		        if (this.pc.remoteDescription.type === 'offer') {
		          this.pc.createAnswer().then(this.localDescCreated).catch(this.onError);
		        }
		      }, this.onError);
		    } else if (message.candidate) {
		      // Add the new ICE candidate to our connections remote description
		      this.pc.addIceCandidate(
		        new RTCIceCandidate(message.candidate), this.onSuccess, this.onError
		      );
		    }
		  });
	  },

	  localDescCreated(desc) {
		  this.pc.setLocalDescription(
		    desc,
		    () => this.sendMessage({'sdp': this.pc.localDescription}),
		    this.onError
		  );
	  },

	  onSuccess() {

	  },

	  onError(error) {
		  console.error(error);
	  },

	  setupDataChannel() {
		  this.checkDataChannelState();
		  this.dataChannel.onopen = this.checkDataChannelState;
		  this.dataChannel.onclose = this.checkDataChannelState;
		  this.dataChannel.onmessage = event =>
		  this.insertMessageToDOM(JSON.parse(event.data), false)
	  },

	  checkDataChannelState() {
		  console.log('WebRTC channel state is:', this.dataChannel.readyState);
		  if (this.dataChannel.readyState === 'open') {
		      this.insertMessageToDOM({content: 'WebRTC data channel is now open'});
		  }
	  },

	  insertMessageToDOM(options, isFromMe) {
	  	  let _name = '';
	  	  let _avatar = '';
	  	  let _sent = false;
	  	  if (isFromMe) {
	  	  	_name = 'me';
	  	  	_avatar = 'https://cdn.quasar.dev/img/avatar1.jpg';
	  	  	_sent = true;
	  	  } else {
	  	  	_name = 'other';
	  	  	_avatar = 'https://cdn.quasar.dev/img/avatar2.jpg';
	  	  }
          this.chat_data.push([_name, _avatar, options.content, _sent]);
          const scrollArea = this.$refs.chatScroll;
		  const scrollTarget = scrollArea.getScrollTarget();
		  const duration = 100; // ms - use 0 to instant scroll
		  scrollArea.setScrollPosition(scrollTarget.scrollHeight, duration);
      },

      sendWord() {
      	  const data = {
		    content: this.editor,
		  };
		  if (!this.dataChannel) {
		  	this.$q.notify({
		        color: 'red-5',
		        textColor: 'white',
		        icon: 'warning',
		        message: 'No one to chat!'
		    });
		    return null;
		  }
  		  this.dataChannel.send(JSON.stringify(data));
  		  this.editor = '';
  		  this.insertMessageToDOM(data, true);
      }

  }
}
</script>
