<template>
  <v-app>
    <v-card class="mx-auto mt-10" width="600px">
      <v-app-bar dark color="primary">
        <v-toolbar-title>
          <v-icon class="ml-4 mr-4">mdi-flask</v-icon>Test your model
        </v-toolbar-title>
      </v-app-bar>
      <v-form ref="predictform" v-model="valid" :lazy-validation="lazy">
        <v-card-text>
          <v-row no-gutters>
            <v-col cols="12" sm="12" lg="12" v-for="(item,key) in features" :key="key">
              <v-text-field
                prepend-icon="mdi-rename-box"
                v-model="item.value"
                :label="item.name"
                required
                :rules="nameRules"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-alert dense text class="success--text" v-show="show">
            <div class="title">
              <v-icon>mdi-auto-fix</v-icon>
              {{target}} : {{prediction}}
            </div>
          </v-alert>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" @click="reset()">Close</v-btn>
          <v-btn color="blue darken-1" :disabled="!valid" @click="add()">Predict</v-btn>
        </v-card-actions>
      </v-form>
    </v-card>
  </v-app>
</template>

<script>
// @ is an alias to /src
import axios from "axios";
export default {
  name: "Home",
  components: {},
  data: () => ({
    valid: true,
    lazy: false,
    features: [],
    target: "",
    nameRules: [v => !!v || "This field is required"],
    model: null,
    prediction: null,
    show: false
  }),
  methods: {
    reset() {
      this.$router.go();
    },
    add() {
      var list = [];
      // var feature = this.features
      for (var item in this.features) {
        // list.push(this.features[item].value);
        switch (this.features[item].datatype) {
          case "float64":
            list.push(parseFloat(this.features[item].value));
            break;

          case "int64":
            list.push(parseInt(this.features[item].value));
            break;
          default:
            break;
        }
      }
      console.log(list);
      var formData = new FormData();
      formData.set("featurelist", list);
      formData.set("modelpath", this.model);
      formData.set("name", this.$route.params.name);
      axios({
        method: "post",
        url: "/api/predictlinear/",
        data: formData,
        config: { headers: { "Content-Type": "multipart/form-data" } }
      }).then(response => {
        console.log(response.data.status);
        this.prediction = response.data.prediction;
        this.show = true;
      });
    }
  },
  created() {
    var dict = JSON.parse(this.$route.params.features);
    for (var key of Object.keys(dict)) {
      var data = {
        name: key,
        value: null,
        datatype: dict[key]
      };
      this.features.push(data);
    }
    this.model = this.$route.params.modelpath;
    this.target = this.$route.params.target;
    // console.log(typeof this.$route.params.features);
    // console.log(this.features);
    // console.log(this.$route.params.features);

    // console.log(this.$route.params.modelpath);
    // console.log(this.$route.params.target);
  }
};
</script>
