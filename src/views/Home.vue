<template>
  <v-app>
    <v-row no-gutters>
      <v-col cols="12" lg="4">
        <v-card class="ml-10 mt-10">
          <v-app-bar dark color="primary">
            <v-toolbar-title>
              <v-icon class="ml-4 mr-4">mdi-flask</v-icon>Train a model
            </v-toolbar-title>
          </v-app-bar>
          <v-form ref="form" v-model="valid" :lazy-validation="lazy">
            <v-card-text>
              <v-row>
                <v-col cols="12" sm="6" md="12">
                  <v-text-field
                    prepend-icon="mdi-rename-box"
                    v-model="model_name"
                    label="Name your model"
                    required
                    :rules="nameRules"
                  ></v-text-field>
                  <v-text-field
                    prepend-icon="mdi-rename-box"
                    v-model="target"
                    label="Enter target variable name"
                    required
                    :rules="nameRules"
                  ></v-text-field>
                  <v-text-field
                    prepend-icon="mdi-rename-box"
                    v-model="test_size"
                    label="Enter test size percentage(suggested 30)"
                    required
                    :rules="percentRules"
                  ></v-text-field>
                  <v-select
                    prepend-icon="mdi-rename-box"
                    v-model="select"
                    :items="items"
                    :rules="nameRules"
                    label="Select algorithm"
                    required
                  ></v-select>
                </v-col>
                <!-- <v-col cols="12" sm="6" md="12">
                      
                </v-col>-->

                <!-- <v-col cols="12" sm="6" md="12">
                      
                </v-col>-->

                <v-col cols="12" sm="6" md="12">
                  <v-file-input
                    v-model="files"
                    color="primary"
                    counter
                    accept="CSV/csv"
                    label="Dataset (CSV file)"
                    placeholder="Enter your dataset"
                    prepend-icon="mdi-paperclip"
                    outlined
                    :show-size="1000"
                    :rules="fileRules"
                  >
                    <template v-slot:selection="{ index, text }">
                      <v-chip v-if="index < 2" color="primary accent-4" dark label small>{{ text }}</v-chip>

                      <span
                        v-else-if="index === 2"
                        class="overline grey--text text--darken-3 mx-2"
                      >+{{ files.length - 2 }} File(s)</span>
                    </template>
                  </v-file-input>
                </v-col>
              </v-row>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" @click="reset()">Close</v-btn>
              <v-btn color="blue darken-1" :disabled="!valid" @click="add()">Train</v-btn>
            </v-card-actions>
          </v-form>
        </v-card>
      </v-col>
      <v-col cols="12" lg="1">
        <v-icon
          v-show="display"
          color="primary"
          size="70"
          style="margin-top: 200%; margin-left:10px;"
        >mdi-arrow-right-bold-circle-outline</v-icon>
      </v-col>
      <v-col cols="12" lg="7">
        <v-card class="mr-2 mt-10" v-show="display" max-width="1200">
          <v-app-bar dark color="primary">
            <v-toolbar-title>
              <v-icon class="ml-4 mr-4">mdi-flask</v-icon>Model metrics
            </v-toolbar-title>
          </v-app-bar>
          <v-row>
            <v-col cols="12" md="6" lg="6" v-for="item in plots" :key="item">
              <v-card class="ml-2" max-width="600">
                <v-img :src="require(`../assets/${item.path}`)" height="300px" width="600px"></v-img>

                <v-card-title>{{item.title}}</v-card-title>

                <v-card-subtitle>{{item.subtitle}}</v-card-subtitle>

                <v-card-actions></v-card-actions>
              </v-card>

              <!-- <v-overlay absolute opacity="0.5" :value="overlay">
                <v-img
                  src="/home/sanfer/Documents/ml-examples-vuejs-flask/web-app/src/assets/matplotlib_histogram.png"
                  height="300px"
                ></v-img>
                <v-btn
                  class="mt-3"
                  style="margin-left: 180px;"
                  fab
                  color="primary lighten-2"
                  @click="overlay = !overlay"
                >
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </v-overlay>-->
            </v-col>
            <v-col cols="12" md="6" lg="6">
              <v-card class="ma-2" max-width="370">
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title class="headline mb-1">Error Metrics</v-list-item-title>

                    <v-list-item v-for="item in metrics" :key="item" two-line>
                      <v-list-item-content>
                        <v-list-item-title>{{item.name}} = {{item.value}}</v-list-item-title>
                        <v-list-item-subtitle>{{item.subtitle}}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                  </v-list-item-content>
                </v-list-item>
              </v-card>
            </v-col>
          </v-row>

          <v-card-actions class="mb-12">
            <v-spacer></v-spacer>
            <v-btn rounded color="primary" dark @click="predict">Make Predictions</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-app>
</template>

<script>
// @ is an alias to /src
import axios from "axios";

export default {
  name: "Home",
  components: {},
  data: () => ({
    overlay: true,
    show: false,
    nameRules: [v => !!v || "Title is required"],
    percentRules: [
      v => /^[1-9][0-9]?$|^100$/.test(v) || "Enter percentage (1-100%)"
    ],
    fileRules: [v => !!v || "File is required"],
    select: null,
    items: [
      "Linear Regression",
      "Lasso Regression",
      "Ridge Regression",
      "Bayesian Ridge Regression"
    ],
    files: null,
    model_name: "",
    target: "",
    test_size: "",
    valid: true,
    lazy: true,
    plots: [],
    metrics: [],
    display: false,
    features: null,
    modelpath: ""
  }),
  methods: {
    predict() {
      this.$router.push({
        name: "About",
        params: {
          features: this.features,
          modelpath: this.modelpath,
          target: this.target,
          name: this.model_name
        }
      });
    },
    add() {
      var formData = new FormData();
      if (this.select == "Linear Regression") {
        formData.set("name", this.model_name);
        formData.set("target", this.target);
        formData.set("test_size", this.test_size / 100);
        formData.set("dataset", this.files);
        axios({
          method: "post",
          url: "/api/LinReg/",
          data: formData,
          config: { headers: { "Content-Type": "multipart/form-data" } }
        })
          .then(response => {
            console.log(response.data.status);
            console.log(response.data.rmse);

            // var data = {
            //   scatterplotpath: response.data.ploturl["scatterplotpath"],
            //   distpath: response.data.ploturl["distpath"],
            //   residualpath: response.data.ploturl["residualpath"]
            // };
            // this.plots = data;-
            var mae = {
              name: "MAE",
              subtitle: "Mean Absolute Error",
              value: response.data.metrics["mae"]
            };
            this.metrics.push(mae);
            var mse = {
              name: "MSE",
              subtitle: "Mean Squared Error",
              value: response.data.metrics["mse"]
            };
            this.metrics.push(mse);
            var rmse = {
              name: "RMSE",
              subtitle: "Root Mean Squared Error",
              value: response.data.metrics["rmse"]
            };
            this.metrics.push(rmse);
            var score = {
              name: "Score",
              subtitle: "R^2 value",
              value: response.data.metrics["r_square"]
            };
            this.metrics.push(score);

            var scatter = {
              path: response.data.ploturl["scatterplotpath"],
              title: "Scatter Plot",
              subtitle: "Predicted vs Actual target variable"
            };
            this.plots.push(scatter);
            var dist = {
              path: response.data.ploturl["distpath"],
              title: "Histogram",
              subtitle: "Of target variable"
            };
            this.plots.push(dist);
            var residual = {
              path: response.data.ploturl["residualpath"],
              title: "Residual Histogram",
              subtitle: "Histogram of Actual-Predicted target variable"
            };
            this.plots.push(residual);
            this.display = true;
            this.features = response.data.feature_names;
            this.modelpath = response.data.model_path;
          })
          .catch(error => {
            console.log(error);
          });
      } else if (this.select == "Lasso Regression") {
        formData.set("name", this.model_name);
        formData.set("target", this.target);
        formData.set("test_size", this.test_size / 100);
        formData.set("dataset", this.files);
        axios({
          method: "post",
          url: "/api/Laso/",
          data: formData,
          config: { headers: { "Content-Type": "multipart/form-data" } }
        })
          .then(response => {
            console.log(response.data.status);
            console.log(response.data.rmse);

            // var data = {
            //   scatterplotpath: response.data.ploturl["scatterplotpath"],
            //   distpath: response.data.ploturl["distpath"],
            //   residualpath: response.data.ploturl["residualpath"]
            // };
            // this.plots = data;-
            var mae = {
              name: "MAE",
              subtitle: "Mean Absolute Error",
              value: response.data.metrics["mae"]
            };
            this.metrics.push(mae);
            var mse = {
              name: "MSE",
              subtitle: "Mean Squared Error",
              value: response.data.metrics["mse"]
            };
            this.metrics.push(mse);
            var rmse = {
              name: "RMSE",
              subtitle: "Root Mean Squared Error",
              value: response.data.metrics["rmse"]
            };
            this.metrics.push(rmse);
            var score = {
              name: "Score",
              subtitle: "R^2 value",
              value: response.data.metrics["r_square"]
            };
            this.metrics.push(score);

            var scatter = {
              path: response.data.ploturl["scatterplotpath"],
              title: "Scatter Plot",
              subtitle: "Predicted vs Actual target variable"
            };
            this.plots.push(scatter);
            var dist = {
              path: response.data.ploturl["distpath"],
              title: "Histogram",
              subtitle: "Of target variable"
            };
            this.plots.push(dist);
            var residual = {
              path: response.data.ploturl["residualpath"],
              title: "Residual Histogram",
              subtitle: "Histogram of Actual-Predicted target variable"
            };
            this.plots.push(residual);
            this.display = true;
            this.features = response.data.feature_names;
            this.modelpath = response.data.model_path;
          })
          .catch(error => {
            console.log(error);
          });
      } else if (this.select == "Ridge Regression") {
        formData.set("name", this.model_name);
        formData.set("target", this.target);
        formData.set("test_size", this.test_size / 100);
        formData.set("dataset", this.files);
        axios({
          method: "post",
          url: "/api/Ridge/",
          data: formData,
          config: { headers: { "Content-Type": "multipart/form-data" } }
        })
          .then(response => {
            console.log(response.data.status);
            console.log(response.data.rmse);

            // var data = {
            //   scatterplotpath: response.data.ploturl["scatterplotpath"],
            //   distpath: response.data.ploturl["distpath"],
            //   residualpath: response.data.ploturl["residualpath"]
            // };
            // this.plots = data;-
            var mae = {
              name: "MAE",
              subtitle: "Mean Absolute Error",
              value: response.data.metrics["mae"]
            };
            this.metrics.push(mae);
            var mse = {
              name: "MSE",
              subtitle: "Mean Squared Error",
              value: response.data.metrics["mse"]
            };
            this.metrics.push(mse);
            var rmse = {
              name: "RMSE",
              subtitle: "Root Mean Squared Error",
              value: response.data.metrics["rmse"]
            };
            this.metrics.push(rmse);
            var score = {
              name: "Score",
              subtitle: "R^2 value",
              value: response.data.metrics["r_square"]
            };
            this.metrics.push(score);

            var scatter = {
              path: response.data.ploturl["scatterplotpath"],
              title: "Scatter Plot",
              subtitle: "Predicted vs Actual target variable"
            };
            this.plots.push(scatter);
            var dist = {
              path: response.data.ploturl["distpath"],
              title: "Histogram",
              subtitle: "Of target variable"
            };
            this.plots.push(dist);
            var residual = {
              path: response.data.ploturl["residualpath"],
              title: "Residual Histogram",
              subtitle: "Histogram of Actual-Predicted target variable"
            };
            this.plots.push(residual);
            this.display = true;
            this.features = response.data.feature_names;
            this.modelpath = response.data.model_path;
          })
          .catch(error => {
            console.log(error);
          });
      } else if (this.select == "Bayesian Ridge Regression") {
        formData.set("name", this.model_name);
        formData.set("target", this.target);
        formData.set("test_size", this.test_size / 100);
        formData.set("dataset", this.files);
        axios({
          method: "post",
          url: "/api/BayReg/",
          data: formData,
          config: { headers: { "Content-Type": "multipart/form-data" } }
        })
          .then(response => {
            console.log(response.data.status);
            console.log(response.data.rmse);

            // var data = {
            //   scatterplotpath: response.data.ploturl["scatterplotpath"],
            //   distpath: response.data.ploturl["distpath"],
            //   residualpath: response.data.ploturl["residualpath"]
            // };
            // this.plots = data;-
            var mae = {
              name: "MAE",
              subtitle: "Mean Absolute Error",
              value: response.data.metrics["mae"]
            };
            this.metrics.push(mae);
            var mse = {
              name: "MSE",
              subtitle: "Mean Squared Error",
              value: response.data.metrics["mse"]
            };
            this.metrics.push(mse);
            var rmse = {
              name: "RMSE",
              subtitle: "Root Mean Squared Error",
              value: response.data.metrics["rmse"]
            };
            this.metrics.push(rmse);
            var score = {
              name: "Score",
              subtitle: "R^2 value",
              value: response.data.metrics["r_square"]
            };
            this.metrics.push(score);

            var scatter = {
              path: response.data.ploturl["scatterplotpath"],
              title: "Scatter Plot",
              subtitle: "Predicted vs Actual target variable"
            };
            this.plots.push(scatter);
            var dist = {
              path: response.data.ploturl["distpath"],
              title: "Histogram",
              subtitle: "Of target variable"
            };
            this.plots.push(dist);
            var residual = {
              path: response.data.ploturl["residualpath"],
              title: "Residual Histogram",
              subtitle: "Histogram of Actual-Predicted target variable"
            };
            this.plots.push(residual);
            this.display = true;
            this.features = response.data.feature_names;
            this.modelpath = response.data.model_path;
          })
          .catch(error => {
            console.log(error);
          });
      }
    },
    reset() {
      this.$router.go();
    }
  }
};
</script>
