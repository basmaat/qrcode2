<template>
  <div>
    <v-stepper v-model="e6" vertical>
      <v-stepper-header>
        <v-stepper-step :complete="e6 > 1" step="1"></v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step :complete="e6 > 2" step="2"></v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step :complete="e6 > 3" step="3"></v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step step="4"></v-stepper-step>
      </v-stepper-header>
      <v-stepper-items>
        <v-stepper-content step="1">Select the store
          <v-card color="grey lighten-1" class="mb-5" height="200px"></v-card>
          <v-btn color="primary" @click.native="e6 = 2">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>

        <v-stepper-content step="2">Take a picture of your coworker
          <v-card color="grey lighten-1" class="mb-5" height="200px"></v-card>
          <v-btn color="primary" @click.native="e6 = 3">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>

        <v-stepper-content step="3">Scan the area
          <v-card color="grey lighten-1" class="mb-5" height="200px">
            <qrcode-stream @decode="onDecode" @init="onInit" />
          </v-card>
          <v-btn color="primary" @click.native="e6 = 4">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>

        <v-stepper-content step="4">Scan the items
          <v-card color="grey lighten-1" class="mb-5" max-width="400" height="200px">
            <qrcode-stream @decode="onDecode" @init="onInit" />
            <v-card-subtitle class="pb-0">
              <v-list rounded>
                <v-subheader>Scanned QR codes</v-subheader>
                <v-list-item-group>
                  <v-list-item v-for="(scan) in results" :key="scan.scan">
                    {{scan.scan}}
                    <v-chip class="ma-2" color="green" text-color="white" close-icon="mdi-delete">
                      <v-avatar left class="green darken-4">{{scan.count}}</v-avatar>Counts
                    </v-chip>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
            </v-card-subtitle>
          </v-card>
          <v-btn color="primary" @click.native="e6 = 1">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>
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
      error: "",
      e6: 1
    };
  },

  methods: {
    onDecode(result) {
      // this.result = result

      //let results = []
      let bas = this.results.findIndex(x => x.scan === result);

      if (bas === -1) {
        this.results.push({
          id: this.results.length + 1,
          scan: result,
          count: 1
        });
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