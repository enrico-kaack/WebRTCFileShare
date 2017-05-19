<template>
  <div id="app">
    <div>{{roomId}}</div>
    <router-view :rtc="rtc" v-on:send="sendFile"></router-view>
  </div>
</template>

<script>
let SimpleWebRTC = require('SimpleWebRTC')

export default {
  name: 'app',
  data() {
      return {
        roomId: '',
        rtc: Object,
        peers: [],
        transfers: []
      }
  },
  created: function () {
    this.initWebRTC();

    this.$data.roomId = this.getRoomId();
    console.log('prarameter room id', this.$data.roomId)
    if (this.$data.roomId == null){
        this.createRoom();
    }else{
        this.joinRoom(this.$data.roomId);
    }



  }, methods: {
      createRoom: function (name) {
        var vm = this;
        this.$data.rtc.createRoom(name, function (err, name) {
          if (err){
            console.log(err);
          }else{
            vm.$data.roomId = name;
            console.log("created room ", name)
          }

        });

      },

      initWebRTC: function () {
        var vm = this;

        this.$data.rtc = new SimpleWebRTC({
          // we don't do video
          localVideoEl: '',
          remoteVideosEl: '',
          debug: true,
          // dont ask for camera access
          autoRequestMedia: false,
          // dont negotiate media
          receiveMedia: {
            offerToReceiveAudio: 0,
            offerToReceiveVideo: 0
          }
        });

        this.$data.rtc.on('createdPeer', this.peerCreated);
        this.$data.rtc.on('fileTransfer', this.incomingFileTransfer);
      },

      joinRoom: function (id) {
        this.$data.rtc.joinRoom(id);
      },

      getRoomId: function () {
        return this.$route.query.roomId;
      },

      sendFile: function (file){
        console.log("peer list", this.$data.peers)
        var vm = this;
        this.$data.peers.forEach(function (curr, index, array) {
            console.log("send for", curr)
          var sender = curr.sendFile(file[0]);
          sender.on('progress', function (bytesSend) {
            console.log(bytesSend)
          })
          sender.on('sentFile', function () {
            console.log('successfull send')
          });
        })





      },

      peerCreated: function (peer) {
        var FileTransfer = require('filetransfer');

        this.$data.peers.push(peer)
        peer.on('channelOpen', function (channel) {
          if (channel.protocol === 'https://simplewebrtc.com/protocol/filetransfer#inband-v1') {
            channel.onmessage = function (event) {
              console.log('inbound message')
              console.log('event data', event.data)
              var metadata = JSON.parse(event.data);
              console.log('metadata', metadata)
              var receiver = new FileTransfer.Receiver();
              receiver.receive(metadata, channel);
              peer.emit('fileTransfer', metadata, receiver);
              receiver.on('receivedFile', function (file, metadata) {
                receiver.channel.close();
              });
            };
          }
        });

        //this.$data.peers = peer
        console.log('Detected new peer', peer)
      },


      incomingFileTransfer: function (metadata, receiver) {
          let v = this;
          let transfer = {
            direction: String,
            totalSize: 0,
            progressedSize: 0,
            fileName: String,
            peerName: String,
            finished: false
          };
          transfer.fileName = metadata.name;
          transfer.totalSize = metadata.size;
          console.log('metadata incoming', metadata)
          this.$data.transfers.push(transfer);
          console.log('incoming filetransfer', receiver, metadata);
          receiver.on('progress', function (bytesReceived) {
            console.log('receive progress', bytesReceived, 'out of', metadata.size);
            transfer.progressedSize = bytesReceived;
            console.log(v.$data.transfers[0].progressedSize)
        });
        // get notified when file is done
        receiver.on('receivedFile', function (file, metadata) {
          console.log('received file', metadata.name, metadata.size);

          // close the channel
          receiver.channel.close();
        });
      }


}

}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
