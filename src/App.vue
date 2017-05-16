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
        peers: []
      }
  },
  created: function () {
    this.initWebRTC();

    this.$data.roomId = this.getRoomId();
    console.log()
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
          console.log(file[0])
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
        this.$data.peers.push(peer)
        //this.$data.peers = peer
        console.log('Detected new peer', peer)
      },

      incomingFileTransfer: function (metadata, receiver) {
        console.log('incoming filetransfer', metadata.name, metadata);
        receiver.on('progress', function (bytesReceived) {
          console.log('receive progress', bytesReceived, 'out of', metadata.size);
        });
        // get notified when file is done
        receiver.on('receivedFile', function (file, metadata) {
          console.log('received file', metadata.name, metadata.size);

          // close the channel
          receiver.channel.close();
        });
        //filelist.appendChild(item);
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
