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
        <v-stepper-content step="1">
          Select the store
          <v-card color="grey lighten-1" class="mb-5" height="200px">
            <v-radio-group v-model="radioGroup">
              <v-radio v-for="store in Store" :key="store.nr" :label="store.name" :value="store.nr"></v-radio>
            </v-radio-group>
          </v-card>
          <v-btn color="primary" @click.native="e6 = 2">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>

        <v-stepper-content step="2">
          Take a picture of your coworker
          <v-card color="grey lighten-1" class="mb-5" height="200px"></v-card>
          <v-btn color="primary" @click.native="e6 = 3">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>
        <v-stepper-content step="3">
          Scan the area
          <v-card color="grey lighten-1" class="mb-5" height="200px">
            <qrcode-stream @decode="onDecode" @init="onInit" />
          </v-card>
          <v-btn color="primary" @click.native="e6 = 4">Continue</v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>
        <v-stepper-content step="4">
          Scan the items
          <v-card color="grey lighten-1" class="mb-5" max-width="400" height="200px">
            <qrcode-stream @decode="onDecode" @init="onInit" />
          </v-card>
          <v-card color="grey lighten-1" class="mb-5" max-width="400" height="200px">
            <v-data-table
              dense
              :headers="headers"
              :items="results"
              :items-per-page="5"
              :sort-by="id"
              :sort-desc="true"
              class="elevation-1"
            ></v-data-table>
          </v-card>

          <v-btn color="primary" @click="addScan()">Finish and Save </v-btn>
          <v-btn flat>Cancel</v-btn>
        </v-stepper-content>
      </v-stepper-items>
    </v-stepper>
  </div>
</template>

<script>
import { QrcodeStream } from "vue-qrcode-reader";
import gql from "graphql-tag";

export const GET_MY_STORES = gql`
  query getMyStores {
    Store {
      nr
      name
      street
      city
    }
  }
`;

export const ADD_SCANS = gql`
  mutation MyMutation($results: [Scan_insert_input!]!) {
  insert_Scan(objects: $results) {
    affected_rows
    returning {
      _id
    }
  }
}
`;

export default {
  components: { QrcodeStream },

  data() {
    return {
      // result: '',
      result: [],
      results: [],
      error: "",
      e6: 1,
      radioGroup: 1,
      Store: [],
      headers: [
        {
          text: "ID",
          align: "start",
          sortable: true,
          value: "id"
        },
        { text: "Scan", value: "scan" },
        { text: "Count", value: "count" }
      ]
    };
  },

  methods: {
    onDecode(result) {
      // this.result = result

      //let results = []
      let bas = this.results.findIndex(x => x.fullscan === result);

      if (bas === -1) {
        //scanned item is not yet in the results
        // this.results.push({                               //Adds it athe end of the array, splice adds itr at the front.
        //   id: this.results.length + 1,
        //   scan: result,
        //   count: 1
        // });
        this.results.splice(0, 0, {
          id: this.results.length + 1,
          scan: result.replace(/(^\w+:|^)\/\//, ""), // takes away any HTTP from the scan
          fullscan: result,
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
    },

    addScan: function () {
      // insert new todo into db
     this.$apollo.mutate({
       mutation: ADD_SCANS,
       variables: {results: this.results},
       update: this.e6 = 1, 
       //(cache, { data: { insert_todos } }) => {
       //  // Read the data from our cache for this query.
       //  // eslint-disable-next-line
       //  console.log(insert_todos);
       //},
     });
    }
   
  },
  apollo: {
    Store: {
      // graphql query
      query: GET_MY_STORES
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