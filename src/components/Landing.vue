<template>
  <div>
    <div>
      <input v-on:change="fileSelected" type="file">
      <button v-on:click="send">Senden</button>
    </div>
    <div>
        <transfer-item v-for="(transfer, index) in transfers" :key="index" :transfer="transfer" :index="index"></transfer-item>

    </div>
  </div>
</template>

<script>
  import TransferItem from "./TransferElement";
  let vm;
export default {
  components: {TransferItem},
  name: 'landing',
  props: ['rtc', 'transfers'],
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
