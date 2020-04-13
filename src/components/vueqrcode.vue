<template>
  <div>
    <v-card class="mx-auto" max-width="400">
      <v-card-title>scan your QR code</v-card-title>

      <qrcode-stream @decode="onDecode" @init="onInit" />

      <v-card-subtitle class="pb-0">
        <v-list rounded>
          <v-subheader>Scanned QR codes</v-subheader>
          <v-list-item-group>
            <v-list-item v-for="(scan) in results" :key="scan.scan">
              {{scan.scan}}
              <v-chip
                class="ma-2"
                color="green"
                text-color="white"
                close-icon="mdi-delete"
                
              >
                <v-avatar left class="green darken-4">{{scan.count}}</v-avatar>Counts
              </v-chip>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-card-subtitle>
    </v-card>
  </div>
</template>

<script>
import { QrcodeStream } from "vue-qrcode-reader";

export default {
  components: { QrcodeStream },

  data() {
    return {
      // result: '',
      result: [],
      results: [],
      error: ""
    };
  },

  methods: {
    onDecode(result) {
      // this.result = result

      //let results = []
      let bas = this.results.findIndex(x => x.scan === result);

      if (bas === -1) {
        this.results.push({ id: this.results.length + 1 , scan: result, count: 1 });
      } else {
        this.results[bas].count = this.results[bas].count + 1;
     //   console.log(this.results, "allready scanned");
      }

      //     this.result.push({scan: result, count: 1});
      //this.result.push(result)
      console.log(result, this.results);
    },

    async onInit(promise) {
      try {
        await promise;
      } catch (error) {
        if (error.name === "NotAllowedError") {
          this.error = "ERROR: you need to grant camera access permisson";
        } else if (error.name === "NotFoundError") {
          this.error = "ERROR: no camera on this device";
        } else if (error.name === "NotSupportedError") {
          this.error = "ERROR: secure context required (HTTPS, localhost)";
        } else if (error.name === "NotReadableError") {
          this.error = "ERROR: is the camera already in use?";
        } else if (error.name === "OverconstrainedError") {
          this.error = "ERROR: installed cameras are not suitable";
        } else if (error.name === "StreamApiNotSupportedError") {
          this.error = "ERROR: Stream API is not supported in this browser";
        }
      }
    }
  }
};
</script>

<style scoped>
.error {
  font-weight: bold;
  color: red;
}
</style>