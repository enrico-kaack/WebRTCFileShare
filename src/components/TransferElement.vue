<template>
    <div>
      <div>{{ transfer.fileName }}</div>
      <progress-bar type="line" ref="bar" :options="options"></progress-bar>
      <div>{{ speedString }}</div>
    </div>
</template>

<script>

    export default {
        name: 'transfer-item', //Name that is used in router
        props: ['transfer', 'index'],
        watch: {
          'transfer.progressedSize': function (value) {
            this.$refs.bar.animate(value / this.$props.transfer.totalSize, {duration: 100})
            this.$refs.bar.setText(this.formatSize(value, this.$props.transfer.totalSize))
          },
          'transfer.performanceChunks': function (value) {
              let speed = 0; //Bytes/second
            value.forEach(function (curr, index, array){
              speed += curr.bytesDelta / (curr.timeDelta/1000);
            });
            speed = speed / value.length;
            this.$data.transferSpeed = speed;
          }
        },
        computed:{
            speedString: function () {
                let category = this.getCategoryForFileSize(this.$data.transferSpeed);
                return this.formatBytes(this.$data.transferSpeed, category) + category.text + "/s"
            }

        },
        data () {
            return {
              transferSpeed: undefined,
              remainingTime: undefined,
              options: {
                strokeWidth: 1,
                from: { color: '#eef441' },
                to: { color: '#8ff442' },
              }
            }
        },
      methods: {
        formatSize: function (processedBytes, totalBytes) {
            let categoryTotal = this.getCategoryForFileSize(totalBytes);
            let categoryCurrent = this.getCategoryForFileSize(processedBytes);

          return this.formatBytes(processedBytes,categoryCurrent) + categoryCurrent.text + "/" + this.formatBytes(totalBytes, categoryTotal) + categoryTotal.text;
        },

        getCategoryForFileSize: function (bytes) {
          let categories = {
            KB: {value: 0, text: 'KB', divider: 1024},
            MB: {value: 1, text: 'MB', divider: 1024*1024},
            GB: {value: 2, text: 'GB', divider: 1024*1024*1024}
          };
          let category;
          //Calculate max size category
          if (bytes / categories.GB.divider > 1){
            category = categories.GB;
          }else if (bytes / categories.MB.divider > 1){
            category = categories.MB;
          }else{
            category = categories.KB;
          }

          return category;
        },
        formatBytes: function (bytes, category) {
          let value = bytes / category.divider;

          //round on second after comma number
          value = value * 100;
          value = Math.round(value);
          value = value /100;

          return value;
        }
      }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
