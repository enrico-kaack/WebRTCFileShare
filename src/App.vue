<template>
  <div id="app">
    <div>{{roomId}}</div>
    <router-view :rtc="rtc" :transfers="transfers" v-on:send="sendFile"></router-view>
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
        let v = this;
        let transfer = {
          direction: String,
          totalSize: 0,
          progressedSize: 0,
          fileName: String,
          peerName: String,
          performanceChunks: [],
          finished: false
        };
        this.$data.peers.forEach(function (curr, index, array) {
          transfer.direction = 'out',
          transfer.fileName = file[0].name;
          transfer.totalSize = file[0].size;
          v.$data.transfers.push(transfer);
          let lastTimeStamp = Date.now();
          var sender = curr.sendFile(file[0]);
          sender.on('progress', function (bytesSend) {
              //add data chunk to performance
              var timeDelta = Date.now() - lastTimeStamp;
              var bytesDelta = bytesSend - transfer.progressedSize;
              lastTimeStamp = Date.now();
              if(transfer.performanceChunks >= 20){
                  transfer.performanceChunks.shift();
              }
              transfer.performanceChunks.push({timeDelta: timeDelta, bytesDelta: bytesDelta});

              transfer.progressedSize = bytesSend;
          })
          sender.on('sentFile', function () {
            transfer.finished = true;
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
            performanceChunks: [],
            finished: false
          };
          transfer.fileName = metadata.name;
          transfer.direction = 'in';
          transfer.totalSize = metadata.size;
          this.$data.transfers.push(transfer);
          let lastTimeStamp = Date.now();
          receiver.on('progress', function (bytesReceived) {
            var timeDelta = Date.now() - lastTimeStamp;
            var bytesDelta = bytesReceived - transfer.progressedSize;
            lastTimeStamp = Date.now();
            if(transfer.performanceChunks.length >= 20){
              transfer.performanceChunks.shift();
            }
            transfer.performanceChunks.push({timeDelta: timeDelta, bytesDelta: bytesDelta});

            transfer.progressedSize = bytesReceived;
        });
        // get notified when file is done
        receiver.on('receivedFile', function (file, metadata) {
          transfer.finished = true;

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
