<template>
    <div>
      <div>{{ transfer.fileName }}</div>
      <progress-bar type="line" ref="bar" class=".progress-bar-striped" :options="options"></progress-bar>

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
          }
        },
        data () {
            return {
              options: {
                strokeWidth: 1,
                from: { color: '#eef441' },
                to: { color: '#8ff442' },
                step: function(state, circle, attachment) {
                  circle.path.setAttribute('stroke', state.color);
                },
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
            KB: {value: 0, text: 'KB', divider: 10E3},
            MB: {value: 1, text: 'MB', divider: 10E6},
            GB: {value: 2, text: 'GB', divider: 10E9}
          };
          let category;
          //Calculate max size category
          if (bytes / 10E9 > 1){
            category = categories.GB;
          }else if (bytes / 10E6 > 1){
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
