<template>
  <div>
    <input v-on:change="fileSelected" type="file">
    <button v-on:click="send">Senden</button>

    <div>


    </div>
  </div>
</template>

<script>
  let vm;
export default {
  name: 'landing',
  props: ['rtc'],
  data () {
    return {
      remoteId: '',
      files: []
    }
  },
  mounted: function () {
    vm = this;
    this.$data.remoteId = this.$route.query.roomId;



  },
  methods: {
    fileSelected(event) {

      this.$data.files.push(event.target.files[0]);
      console.log(this.$data.files)
    },
    send(){
      console.log("send")
      this.$emit('send', [this.$data.files[0]]);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
